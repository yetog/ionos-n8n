# IONOS AI Model Hub + n8n Integration Guide

🚀 **Complete guide for integrating IONOS AI Model Hub with n8n for Retrieval-Augmented Generation (RAG) workflows**

## 📋 Overview

This repository provides everything you need to integrate IONOS AI Model Hub with n8n to create powerful AI-driven workflows, including RAG (Retrieval-Augmented Generation) systems that combine your documents with AI models.

### 🎯 What You'll Build
- Vector database collections for document storage
- Semantic search across your documents
- OpenAI-compatible API integration
- Complete RAG pipeline in n8n
- European GDPR-compliant AI workflows

## 🏁 Quick Start

### Prerequisites
- IONOS Cloud account with AI Model Hub access
- n8n instance (self-hosted or cloud)
- IONOS API authentication token

### 1️⃣ Get Your IONOS API Token
1. Login to [IONOS Cloud Dashboard](https://cloud.ionos.com)
2. Go to **Access Management** → **API Tokens**
3. Create new token with AI Model Hub permissions
4. Copy the JWT token (starts with `eyJ`)

### 2️⃣ Configure n8n Environment
```bash
# Add to your n8n environment
export IONOS_API_KEY="your_jwt_token_here"

# Restart n8n with environment variable
docker run -d --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e IONOS_API_KEY="$IONOS_API_KEY" \
  n8nio/n8n:latest
```

### 3️⃣ Import the Workflow
1. Download `ionos_rag_workflow.json`
2. Open n8n interface
3. **Settings** → **Import from File**
4. Select the JSON file and import
5. Execute the workflow!

## 🔧 Available Models

IONOS AI Model Hub provides access to leading open-source models:

| Model | ID | Use Case |
|-------|----|---------|
| **Llama 3.3 70B** | `meta-llama/Llama-3.3-70B-Instruct` | General chat, RAG |
| **Llama 3.1 405B** | `meta-llama/Meta-Llama-3.1-405B-Instruct-FP8` | Complex reasoning |
| **Mistral Small** | `mistralai/Mistral-Small-24B-Instruct` | Fast responses |
| **CodeLlama** | `meta-llama/CodeLlama-13b-Instruct-hf` | Code generation |
| **BGE Embeddings** | `BAAI/bge-large-en-v1.5` | Text embeddings |
| **FLUX Image** | `black-forest-labs/FLUX.1-schnell` | Image generation |

## 📚 Documentation Structure

```
/docs/
├── getting-started.md      # Step-by-step setup guide
├── api-reference.md        # Complete API documentation
├── workflow-examples/      # Example n8n workflows
├── troubleshooting.md      # Common issues & solutions
└── best-practices.md       # Performance & security tips

/workflows/
├── basic-chat.json         # Simple chat completion
├── rag-pipeline.json       # Complete RAG workflow
├── document-processing.json # Bulk document ingestion
└── multi-model.json        # Using multiple AI models

/examples/
├── python-scripts/         # Python integration examples
├── curl-commands.md        # API testing with curl
└── postman-collection.json # Postman API collection
```

## 🌟 Key Features

### ✅ **European Data Sovereignty**
- GDPR compliant AI processing
- European data centers only
- No data sent to external providers

### ✅ **OpenAI Compatible API**
- Drop-in replacement for OpenAI
- Same request/response format
- Easy migration from existing workflows

### ✅ **Vector Database Integration**
- PgVector and ChromaDB support
- Automatic document chunking
- Semantic search capabilities

### ✅ **n8n Native Integration**
- Pre-built workflow templates
- Environment variable support
- Error handling and retry logic

## 🔗 API Endpoints

| Service | Base URL |
|---------|----------|
| **Main API** | `https://inference.de-txl.ionos.com` |
| **OpenAI Compatible** | `https://openai.inference.de-txl.ionos.com/v1` |

### Authentication
```bash
# All requests require Bearer token authentication
Authorization: Bearer YOUR_IONOS_JWT_TOKEN
```

## 🚦 Getting Started Workflows

### 1. Simple Chat Completion
```json
POST https://openai.inference.de-txl.ionos.com/v1/chat/completions
{
  "model": "meta-llama/Llama-3.3-70B-Instruct",
  "messages": [{"role": "user", "content": "Hello!"}]
}
```

### 2. Create Vector Collection
```json
POST https://inference.de-txl.ionos.com/collections
{
  "name": "my-documents",
  "engine": {"db_type": "pgvector"}
}
```

### 3. Add Document
```json
POST https://inference.de-txl.ionos.com/collections/{id}/documents
{
  "content": "Your document text here"
}
```

### 4. Query with RAG
```json
POST https://inference.de-txl.ionos.com/collections/{id}/query
{
  "query": "What is this document about?",
  "top_k": 3
}
```

## 🎓 Learning Path

1. **[Getting Started](docs/getting-started.md)** - Basic setup and first workflow
2. **[API Reference](docs/api-reference.md)** - Complete API documentation
3. **[Workflow Examples](workflows/)** - Ready-to-use n8n workflows
4. **[Best Practices](docs/best-practices.md)** - Optimization and security

## 🔧 Troubleshooting

### Common Issues

**401 Unauthorized**
- Check your API token is correct
- Ensure token has AI Model Hub permissions
- Verify token hasn't expired

**Collection Not Found**
- Collection creation takes a few seconds
- Check collection ID is correct
- Verify collection exists before querying

**Rate Limits**
- IONOS has reasonable rate limits
- Implement retry logic in workflows
- Contact support for higher limits

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📧 Email: support@360foresight.info
- 📚 IONOS Docs: https://docs.ionos.com/cloud/ai/ai-model-hub
- 💬 n8n Community: https://community.n8n.io

---

**Built with ❤️ for the European AI community**

*This project demonstrates how to leverage European AI infrastructure for compliant, powerful automation workflows.*