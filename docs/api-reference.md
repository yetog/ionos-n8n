# 📖 IONOS AI Model Hub API Reference

Complete reference for all IONOS AI Model Hub APIs with examples for n8n integration.

## 🔐 Authentication

All API requests require Bearer token authentication:

```bash
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

## 🌐 Base URLs

| Service | URL |
|---------|-----|
| **AI Model Hub API** | `https://inference.de-txl.ionos.com` |
| **OpenAI Compatible API** | `https://openai.inference.de-txl.ionos.com/v1` |

## 🤖 Models

### List Available Models

#### Native API
```http
GET https://inference.de-txl.ionos.com/models
```

#### OpenAI Compatible
```http
GET https://openai.inference.de-txl.ionos.com/v1/models
```

### Available Models

| Model | ID | Category | Use Case |
|-------|----|----------|----------|
| **Llama 3.3 70B** | `meta-llama/Llama-3.3-70B-Instruct` | Text/NLP | General chat, RAG |
| **Llama 3.1 405B** | `meta-llama/Meta-Llama-3.1-405B-Instruct-FP8` | Text/NLP | Complex reasoning |
| **Llama 3.1 8B** | `meta-llama/Meta-Llama-3.1-8B-Instruct` | Text/NLP | Fast responses |
| **Mistral Small** | `mistralai/Mistral-Small-24B-Instruct` | Text/NLP | Efficient processing |
| **Mistral Nemo** | `mistralai/Mistral-Nemo-Instruct-2407` | Text/NLP | Multilingual |
| **CodeLlama 13B** | `meta-llama/CodeLlama-13b-Instruct-hf` | Text/NLP | Code generation |
| **Teuken 7B** | `openGPT-X/Teuken-7B-instruct-commercial` | Text/NLP | German language |
| **GPT OSS 120B** | `openai/gpt-oss-120b` | Text/NLP | Advanced reasoning |
| **BGE-M3** | `BAAI/bge-m3` | Text/NLP | Multilingual embeddings |
| **BGE Large** | `BAAI/bge-large-en-v1.5` | Text/NLP | English embeddings |
| **Paraphrase MPNet** | `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` | Text/NLP | Multilingual embeddings |
| **FLUX Schnell** | `black-forest-labs/FLUX.1-schnell` | Image/Generation | Fast image generation |

## 💬 Chat Completions

### OpenAI Compatible Endpoint

```http
POST https://openai.inference.de-txl.ionos.com/v1/chat/completions
```

#### Request Body
```json
{
  "model": "meta-llama/Llama-3.3-70B-Instruct",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "Hello, how are you?"
    }
  ],
  "max_tokens": 150,
  "temperature": 0.7,
  "top_p": 0.9,
  "stream": false
}
```

#### n8n HTTP Request Node Configuration
```json
{
  "url": "https://openai.inference.de-txl.ionos.com/v1/chat/completions",
  "method": "POST",
  "headers": {
    "Authorization": "Bearer {{ $env.IONOS_API_KEY }}",
    "Content-Type": "application/json"
  },
  "body": {
    "model": "meta-llama/Llama-3.3-70B-Instruct",
    "messages": "{{ $json.messages }}",
    "max_tokens": 150
  }
}
```

#### Response
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1699999999,
  "model": "meta-llama/Llama-3.3-70B-Instruct",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! I'm doing well, thank you for asking."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 12,
    "total_tokens": 37
  }
}
```

## 📚 Collections (Vector Database)

### Create Collection

```http
POST https://inference.de-txl.ionos.com/collections
```

#### Request Body
```json
{
  "name": "my-knowledge-base",
  "description": "Company documents and policies",
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

#### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Collection name (unique) |
| `description` | string | Optional description |
| `engine.db_type` | string | `pgvector` or `chromadb` |
| `chunking.enabled` | boolean | Auto-chunk documents |
| `chunking.strategy.name` | string | `fixed_size` or `semantic` |
| `chunking.strategy.config.chunk_size` | integer | Characters per chunk |
| `chunking.strategy.config.chunk_overlap` | integer | Overlap between chunks |

#### Response
```json
{
  "id": "collection-abc123",
  "name": "my-knowledge-base",
  "description": "Company documents and policies",
  "status": "creating",
  "created_at": "2024-01-01T12:00:00Z"
}
```

### List Collections

```http
GET https://inference.de-txl.ionos.com/collections
```

### Get Collection

```http
GET https://inference.de-txl.ionos.com/collections/{collection_id}
```

### Delete Collection

```http
DELETE https://inference.de-txl.ionos.com/collections/{collection_id}
```

## 📄 Documents

### Add Document to Collection

```http
POST https://inference.de-txl.ionos.com/collections/{collection_id}/documents
```

#### Request Body
```json
{
  "content": "This is the document content that will be searchable...",
  "metadata": {
    "source": "company-policy.pdf",
    "type": "policy",
    "department": "HR",
    "date": "2024-01-01"
  }
}
```

#### Response
```json
{
  "id": "doc-abc123",
  "collection_id": "collection-abc123",
  "status": "processing",
  "chunks_created": 5,
  "created_at": "2024-01-01T12:00:00Z"
}
```

### List Documents

```http
GET https://inference.de-txl.ionos.com/collections/{collection_id}/documents
```

### Get Document

```http
GET https://inference.de-txl.ionos.com/collections/{collection_id}/documents/{document_id}
```

### Delete Document

```http
DELETE https://inference.de-txl.ionos.com/collections/{collection_id}/documents/{document_id}
```

## 🔍 Query Collection

### Semantic Search

```http
POST https://inference.de-txl.ionos.com/collections/{collection_id}/query
```

#### Request Body
```json
{
  "query": "What is the company vacation policy?",
  "top_k": 5,
  "filter": {
    "metadata.type": "policy",
    "metadata.department": "HR"
  },
  "include_metadata": true
}
```

#### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | string | Search query text |
| `top_k` | integer | Number of results (default: 10) |
| `filter` | object | Metadata filters |
| `include_metadata` | boolean | Include document metadata |

#### Response
```json
{
  "documents": [
    {
      "id": "chunk-abc123",
      "content": "Company vacation policy allows 25 days...",
      "score": 0.95,
      "metadata": {
        "source": "hr-policy.pdf",
        "type": "policy",
        "page": 5
      }
    }
  ],
  "query": "What is the company vacation policy?",
  "total_results": 3
}
```

## 🔤 Text Embeddings

### Generate Embeddings

```http
POST https://openai.inference.de-txl.ionos.com/v1/embeddings
```

#### Request Body
```json
{
  "model": "BAAI/bge-large-en-v1.5",
  "input": [
    "Text to embed",
    "Another text to embed"
  ]
}
```

#### Response
```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [0.1, 0.2, -0.3, ...]
    }
  ],
  "model": "BAAI/bge-large-en-v1.5",
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```

## 🖼️ Image Generation

### Generate Image

```http
POST https://inference.de-txl.ionos.com/models/flux-1-schnell/predictions
```

#### Request Body
```json
{
  "input": {
    "prompt": "A beautiful sunset over mountains",
    "width": 1024,
    "height": 1024,
    "num_inference_steps": 4
  }
}
```

## 🔧 n8n Integration Examples

### HTTP Request Node Template

```json
{
  "parameters": {
    "url": "https://openai.inference.de-txl.ionos.com/v1/chat/completions",
    "method": "POST",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "Bearer {{ $env.IONOS_API_KEY }}"
        },
        {
          "name": "Content-Type",
          "value": "application/json"
        }
      ]
    },
    "sendBody": true,
    "contentType": "json",
    "body": {
      "model": "meta-llama/Llama-3.3-70B-Instruct",
      "messages": [
        {
          "role": "user",
          "content": "{{ $json.query }}"
        }
      ]
    }
  },
  "name": "IONOS Chat",
  "type": "n8n-nodes-base.httpRequest"
}
```

### Error Handling

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "{{ $node['IONOS Chat'].json.error }}",
          "operation": "isNotEmpty"
        }
      ]
    }
  },
  "name": "Check for Errors",
  "type": "n8n-nodes-base.if"
}
```

## 📊 Rate Limits

| Endpoint | Limit | Time Window |
|----------|-------|-------------|
| Chat Completions | 100 req/min | Per model |
| Embeddings | 200 req/min | Per model |
| Collections | 50 req/min | Total |
| Documents | 100 req/min | Total |

## 🐛 Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| `401` | Unauthorized | Check API token |
| `404` | Not Found | Verify endpoint/resource exists |
| `429` | Rate Limited | Implement retry with backoff |
| `500` | Server Error | Check IONOS status page |

## 💡 Best Practices

1. **Use environment variables** for API tokens
2. **Implement retry logic** for rate limits
3. **Cache responses** when appropriate
4. **Use specific models** for different tasks
5. **Add metadata** to documents for better search
6. **Monitor usage** in IONOS dashboard
7. **Test with small datasets** first

---

**Need help?** Check the [troubleshooting guide](troubleshooting.md) or [n8n community](https://community.n8n.io).