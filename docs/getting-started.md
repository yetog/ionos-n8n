# 🚀 Getting Started with IONOS AI Model Hub + n8n

This guide will walk you through setting up your first IONOS AI Model Hub workflow in n8n, from zero to a working RAG (Retrieval-Augmented Generation) system.

## 📋 Prerequisites

Before you begin, make sure you have:

- [ ] IONOS Cloud account with AI Model Hub access
- [ ] n8n instance (self-hosted or n8n Cloud)
- [ ] Basic understanding of APIs and workflows
- [ ] 15 minutes of your time

## 🎯 What We'll Build

By the end of this tutorial, you'll have a working workflow that:
1. Creates a vector database collection
2. Adds documents to the collection
3. Queries the collection with semantic search
4. Uses AI to answer questions based on your documents

## Step 1: Get Your IONOS API Token

### 1.1 Login to IONOS Cloud
1. Go to [cloud.ionos.com](https://cloud.ionos.com)
2. Sign in with your IONOS account
3. Navigate to **Access Management**

### 1.2 Create API Token
1. Click **API Tokens** in the left sidebar
2. Click **+ Create Token**
3. Give it a name like "n8n-ai-hub-token"
4. Select permissions:
   - ✅ `ACCESS_AND_MANAGE_AI_MODEL_HUB`
   - ✅ `MANAGE_DATAPLATFORM` (optional, for advanced features)
5. Click **Create Token**
6. **IMPORTANT**: Copy the JWT token immediately (starts with `eyJ`)

### 1.3 Test Your Token
```bash
# Test your token works
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" \
     https://inference.de-txl.ionos.com/models
```

You should see a list of available AI models.

## Step 2: Configure n8n

### 2.1 For Self-Hosted n8n (Docker)
```bash
# Stop existing n8n container
docker stop n8n
docker rm n8n

# Start n8n with IONOS token
docker run -d --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e IONOS_API_KEY="YOUR_JWT_TOKEN_HERE" \
  n8nio/n8n:latest
```

### 2.2 For n8n Cloud
1. Go to your n8n Cloud instance
2. **Settings** → **Environments**
3. Add new environment variable:
   - Name: `IONOS_API_KEY`
   - Value: Your JWT token
4. Save and restart your instance

### 2.3 Verify Configuration
1. Open your n8n interface
2. Create a new workflow
3. Add an **HTTP Request** node
4. In the headers, try using `{{ $env.IONOS_API_KEY }}`
5. It should resolve to your token value

## Step 3: Import the RAG Workflow

### 3.1 Download the Workflow
Download the `rag-pipeline.json` from this repository or copy the JSON from the workflow section.

### 3.2 Import to n8n
1. In n8n, click the **Settings** (gear icon)
2. Select **Import from File**
3. Choose the `rag-pipeline.json` file
4. Click **Import**

### 3.3 Review the Workflow
The imported workflow has 5 nodes:
1. **Manual Trigger** - Starts the workflow
2. **Create Collection** - Creates vector database
3. **Add Document** - Adds sample content
4. **Query Collection** - Searches for relevant content
5. **Ask Knowledge Base** - Uses AI to answer questions

## Step 4: Execute Your First RAG Workflow

### 4.1 Execute the Workflow
1. Click the **Manual Trigger** node
2. Click **Execute Workflow**
3. Watch each node execute in sequence

### 4.2 Check the Results
1. Click on the **Ask Knowledge Base** node
2. View the output in the right panel
3. You should see an AI response about IONOS features

### 4.3 Customize the Content
1. Click the **Add Document** node
2. Modify the `content` field with your own document
3. Update the question in **Ask Knowledge Base** node
4. Execute again to see results with your content

## Step 5: Understanding the Workflow

### 5.1 Create Collection Node
```json
{
  "name": "Internal_Policy_Beta_{{ $now }}",
  "description": "Beta testing collection",
  "engine": {
    "db_type": "pgvector"
  },
  "chunking": {
    "enabled": true,
    "strategy": {
      "name": "fixed_size",
      "config": {
        "chunk_size": 512,
        "chunk_overlap": 50
      }
    }
  }
}
```
- Creates a new vector database collection
- Uses PgVector as the database engine
- Automatically chunks documents into 512-character pieces

### 5.2 Add Document Node
```json
{
  "content": "Your document text here...",
  "metadata": {
    "source": "filename.pdf",
    "type": "policy_document"
  }
}
```
- Adds content to the collection
- Automatically generates embeddings
- Metadata helps with filtering

### 5.3 Query Collection Node
```json
{
  "query": "What are the key features?",
  "top_k": 3
}
```
- Performs semantic search
- Returns top 3 most relevant chunks
- Uses vector similarity matching

### 5.4 Ask Knowledge Base Node
```json
{
  "model": "meta-llama/Llama-3.3-70B-Instruct",
  "messages": [
    {
      "role": "system",
      "content": "Use this context: {{ $node['Query Collection'].json['documents'][0]['content'] }}"
    },
    {
      "role": "user",
      "content": "Answer based on the documents"
    }
  ]
}
```
- Uses OpenAI-compatible API
- Combines retrieved context with user question
- Returns AI-generated answer

## 🎉 Congratulations!

You've successfully created your first IONOS AI Model Hub RAG workflow!

## 🔄 Next Steps

1. **[Try Different Models](api-reference.md#models)** - Experiment with Mistral, CodeLlama, etc.
2. **[Add More Documents](workflow-examples/)** - Build a larger knowledge base
3. **[Optimize Performance](best-practices.md)** - Learn tuning techniques
4. **[Handle Errors](troubleshooting.md)** - Robust error handling

## 🆘 Need Help?

- Check the [Troubleshooting Guide](troubleshooting.md)
- Review [API Reference](api-reference.md)
- Ask questions in [n8n Community](https://community.n8n.io)

## 💡 Pro Tips

- **Use meaningful collection names** with timestamps
- **Add metadata** to documents for better organization
- **Test with small documents** first before processing large files
- **Monitor your API usage** in the IONOS dashboard
- **Use environment variables** for sensitive data

---

**Ready for more advanced workflows?** Check out our [Workflow Examples](../workflows/) for inspiration!