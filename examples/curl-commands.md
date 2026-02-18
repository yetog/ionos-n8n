# 🧪 IONOS AI Model Hub API Testing with curl

This guide provides curl commands to test all IONOS AI Model Hub endpoints directly from the command line.

## 🔐 Setup

First, export your IONOS API token:

```bash
export IONOS_API_KEY="your_jwt_token_here"
```

## 🤖 Models

### List Available Models

#### Native API
```bash
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/models
```

#### OpenAI Compatible
```bash
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://openai.inference.de-txl.ionos.com/v1/models
```

## 💬 Chat Completions

### Simple Chat
```bash
curl -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.3-70B-Instruct",
    "messages": [
      {
        "role": "user",
        "content": "Hello! Tell me about IONOS AI Model Hub."
      }
    ],
    "max_tokens": 150
  }'
```

### Chat with System Message
```bash
curl -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.3-70B-Instruct",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful European AI assistant focused on data privacy."
      },
      {
        "role": "user",
        "content": "What are the benefits of using European AI services?"
      }
    ],
    "max_tokens": 200,
    "temperature": 0.7
  }'
```

### Code Generation
```bash
curl -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/CodeLlama-13b-Instruct-hf",
    "messages": [
      {
        "role": "user",
        "content": "Write a Python function to validate email addresses using regex."
      }
    ],
    "max_tokens": 300
  }'
```

## 📚 Collections (Vector Database)

### Create Collection
```bash
curl -X POST https://inference.de-txl.ionos.com/collections \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "test-collection-'$(date +%s)'",
    "description": "Test collection for curl examples",
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
  }'
```

### List Collections
```bash
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/collections
```

### Get Specific Collection
```bash
# Replace COLLECTION_ID with actual ID from create response
COLLECTION_ID="your-collection-id"

curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/collections/$COLLECTION_ID
```

### Delete Collection
```bash
curl -X DELETE \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  https://inference.de-txl.ionos.com/collections/$COLLECTION_ID
```

## 📄 Documents

### Add Document
```bash
# Use collection ID from previous step
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "IONOS Cloud is a European Infrastructure as a Service provider. We offer AI Model Hub with leading open-source models like Llama, Mistral, and CodeLlama. Our services are GDPR compliant and hosted in European data centers. Key features include vector databases, OpenAI-compatible APIs, and semantic search capabilities for building RAG applications.",
    "metadata": {
      "source": "ionos-overview.txt",
      "type": "documentation",
      "category": "product-info",
      "date": "'$(date -I)'"
    }
  }'
```

### Add Multiple Documents
```bash
# Document 1
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "IONOS AI Model Hub pricing is competitive and transparent. We offer pay-per-use pricing for API calls and vector database storage. Enterprise customers can get dedicated resources and custom pricing. The discovery offer is free until March 31, 2025.",
    "metadata": {
      "source": "pricing-info.txt",
      "type": "pricing",
      "category": "business-info"
    }
  }'

# Document 2
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Getting started with IONOS AI Model Hub is simple. First, create an account and generate an API token. Then choose your AI model and make API calls using the OpenAI-compatible endpoints. Vector databases can be created for RAG applications.",
    "metadata": {
      "source": "getting-started.txt",
      "type": "tutorial",
      "category": "documentation"
    }
  }'
```

### List Documents in Collection
```bash
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents
```

## 🔍 Query Collection

### Simple Query
```bash
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/query \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is IONOS AI Model Hub?",
    "top_k": 3
  }'
```

### Query with Filters
```bash
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/query \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "How much does it cost?",
    "top_k": 2,
    "filter": {
      "metadata.type": "pricing"
    },
    "include_metadata": true
  }'
```

### Query for Technical Info
```bash
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/query \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "How do I get started with the API?",
    "top_k": 5,
    "filter": {
      "metadata.category": "documentation"
    }
  }'
```

## 🔤 Text Embeddings

### Generate Embeddings (Single Text)
```bash
curl -X POST https://openai.inference.de-txl.ionos.com/v1/embeddings \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "BAAI/bge-large-en-v1.5",
    "input": "IONOS AI Model Hub provides European AI services"
  }'
```

### Generate Embeddings (Multiple Texts)
```bash
curl -X POST https://openai.inference.de-txl.ionos.com/v1/embeddings \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "BAAI/bge-large-en-v1.5",
    "input": [
      "IONOS provides AI services in Europe",
      "Vector databases enable semantic search",
      "OpenAI compatible APIs are easy to use"
    ]
  }'
```

### Multilingual Embeddings
```bash
curl -X POST https://openai.inference.de-txl.ionos.com/v1/embeddings \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sentence-transformers/paraphrase-multilingual-mpnet-base-v2",
    "input": [
      "IONOS bietet KI-Dienste in Europa",
      "IONOS offers AI services in Europe",
      "IONOS propose des services IA en Europe"
    ]
  }'
```

## 🔄 Complete RAG Pipeline

### Full RAG Example
```bash
# 1. Create collection
COLLECTION_RESPONSE=$(curl -s -X POST https://inference.de-txl.ionos.com/collections \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "rag-demo-'$(date +%s)'",
    "description": "RAG demo collection",
    "engine": {"db_type": "pgvector"},
    "chunking": {"enabled": true}
  }')

COLLECTION_ID=$(echo $COLLECTION_RESPONSE | grep -o '"id":"[^"]*' | cut -d'"' -f4)
echo "Created collection: $COLLECTION_ID"

# 2. Add document
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "IONOS AI Model Hub is a fully managed AI platform that provides access to leading open-source AI models. It offers GDPR-compliant processing, European data sovereignty, vector databases for RAG applications, and OpenAI-compatible APIs. Popular models include Llama 3.3 70B, Mistral, CodeLlama, and various embedding models."
  }' > /dev/null

echo "Document added, waiting for processing..."
sleep 5

# 3. Query the collection
QUERY_RESPONSE=$(curl -s -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/query \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What models are available in IONOS AI Model Hub?",
    "top_k": 1
  }')

# Extract the content for RAG context
CONTEXT=$(echo $QUERY_RESPONSE | grep -o '"content":"[^"]*' | cut -d'"' -f4)
echo "Retrieved context: $CONTEXT"

# 4. Generate AI response with context
curl -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.3-70B-Instruct",
    "messages": [
      {
        "role": "system",
        "content": "Use this context to answer questions: '"$CONTEXT"'"
      },
      {
        "role": "user",
        "content": "Based on the information provided, what AI models does IONOS offer?"
      }
    ],
    "max_tokens": 200
  }'

echo -e "\n\nRAG pipeline completed!"
```

## 🧪 Testing Scripts

### Save as `test-ionos-api.sh`
```bash
#!/bin/bash

# Test script for IONOS AI Model Hub API
set -e

echo "🧪 Testing IONOS AI Model Hub API"
echo "=================================="

# Check if API key is set
if [ -z "$IONOS_API_KEY" ]; then
    echo "❌ IONOS_API_KEY not set. Export your token first:"
    echo "   export IONOS_API_KEY=\"your_token_here\""
    exit 1
fi

echo "✅ API key configured"

# Test 1: List models
echo "🤖 Testing model list..."
curl -s -H "Authorization: Bearer $IONOS_API_KEY" \
     https://openai.inference.de-txl.ionos.com/v1/models | \
     head -3
echo "✅ Models endpoint working"

# Test 2: Simple chat
echo "💬 Testing chat completion..."
CHAT_RESPONSE=$(curl -s -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.3-70B-Instruct",
    "messages": [{"role": "user", "content": "Say hello!"}],
    "max_tokens": 50
  }')

if echo "$CHAT_RESPONSE" | grep -q "choices"; then
    echo "✅ Chat completion working"
else
    echo "❌ Chat completion failed"
    echo "$CHAT_RESPONSE"
fi

echo "🎉 All tests passed!"
```

### Make it executable
```bash
chmod +x test-ionos-api.sh
./test-ionos-api.sh
```

## 📊 Response Analysis

### Pretty Print JSON Responses
```bash
# Install jq if not available
# Ubuntu/Debian: sudo apt-get install jq
# macOS: brew install jq

# Use jq to format responses
curl -s -H "Authorization: Bearer $IONOS_API_KEY" \
     https://openai.inference.de-txl.ionos.com/v1/models | jq '.'
```

### Extract Specific Fields
```bash
# Get just model names
curl -s -H "Authorization: Bearer $IONOS_API_KEY" \
     https://openai.inference.de-txl.ionos.com/v1/models | \
     jq -r '.data[].id'

# Get chat response content only
curl -s -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "meta-llama/Llama-3.3-70B-Instruct", "messages": [{"role": "user", "content": "Hello"}], "max_tokens": 50}' | \
  jq -r '.choices[0].message.content'
```

## 🚨 Error Handling

### Check for Errors in Responses
```bash
response=$(curl -s -X POST https://openai.inference.de-txl.ionos.com/v1/chat/completions \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "invalid-model", "messages": [{"role": "user", "content": "test"}]}')

if echo "$response" | grep -q "error"; then
    echo "❌ Error occurred:"
    echo "$response" | jq '.error'
else
    echo "✅ Success:"
    echo "$response" | jq '.choices[0].message.content'
fi
```

---

**Need help?** Check the [API Reference](../docs/api-reference.md) or [troubleshooting guide](../docs/troubleshooting.md).