# 🏆 Best Practices for IONOS AI Model Hub + n8n

Optimize performance, security, and reliability in your AI workflows.

## 🔐 Security Best Practices

### API Token Management

#### ✅ DO
```bash
# Use environment variables
export IONOS_API_KEY="your_token"
docker run -e IONOS_API_KEY n8n

# In n8n workflows
Authorization: Bearer {{ $env.IONOS_API_KEY }}
```

#### ❌ DON'T
```json
// Never hardcode tokens in workflows
{
  "Authorization": "Bearer eyJ0eXAiOiJKV1QiLCJra..."  // ❌ Security risk
}
```

### Token Rotation
```bash
# Set up automated token rotation
#!/bin/bash
# rotate-ionos-token.sh
NEW_TOKEN=$(curl -X POST https://api.ionos.com/auth/tokens \
  -H "Authorization: Bearer $OLD_TOKEN" \
  -d '{"name": "rotated-token-'$(date +%Y%m%d)'"}')

export IONOS_API_KEY="$NEW_TOKEN"
docker restart n8n
```

### Permissions
- Use **principle of least privilege**
- Only grant `ACCESS_AND_MANAGE_AI_MODEL_HUB` if needed
- Create separate tokens for different environments (dev/staging/prod)

## ⚡ Performance Optimization

### Model Selection Strategy

#### For Different Use Cases
```javascript
// In n8n Code node - dynamic model selection
const task = $input.first().json.task_type;
const models = {
  'simple_chat': 'meta-llama/Meta-Llama-3.1-8B-Instruct',      // Fast
  'complex_reasoning': 'meta-llama/Llama-3.3-70B-Instruct',    // Powerful
  'code_generation': 'meta-llama/CodeLlama-13b-Instruct-hf',   // Specialized
  'multilingual': 'mistralai/Mistral-Nemo-Instruct-2407'       // Languages
};

return [{ json: { model: models[task] || models.simple_chat } }];
```

### Token Optimization

#### Efficient Prompting
```json
// ✅ Concise and specific
{
  "messages": [
    {
      "role": "system",
      "content": "Answer in 2-3 sentences."
    },
    {
      "role": "user",
      "content": "What is IONOS AI Model Hub?"
    }
  ],
  "max_tokens": 100
}
```

#### ❌ Inefficient Prompting
```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are an extremely helpful and knowledgeable assistant with deep expertise in cloud computing, artificial intelligence, machine learning, and European data regulations..."  // Too verbose
    }
  ],
  "max_tokens": 2000  // Unnecessarily high
}
```

### Caching Strategy

#### Response Caching
```javascript
// In n8n Code node
const crypto = require('crypto');
const input = $input.first().json;
const cacheKey = crypto.createHash('md5')
  .update(JSON.stringify(input))
  .digest('hex');

// Check cache first (implement with Redis/database)
const cached = await checkCache(cacheKey);
if (cached) {
  return [{ json: cached }];
}

// Cache the response after API call
// await saveToCache(cacheKey, response, ttl: 3600);
```

#### Collection Reuse
```javascript
// Reuse existing collections instead of creating new ones
const existingCollections = await getCollections();
const collection = existingCollections.find(c =>
  c.name.includes('knowledge-base') &&
  isRecent(c.created_at, '1 day')
);

if (collection) {
  return [{ json: { collection_id: collection.id } }];
}
// Only create new collection if needed
```

### Parallel Processing

#### Concurrent API Calls
```json
// n8n workflow - parallel model comparison
{
  "connections": {
    "Manual Trigger": {
      "main": [
        [
          { "node": "Llama 3.3 70B", "type": "main", "index": 0 },
          { "node": "Mistral Small", "type": "main", "index": 0 },
          { "node": "CodeLlama", "type": "main", "index": 0 }
        ]
      ]
    }
  }
}
```

## 📚 RAG Optimization

### Document Chunking Strategy

#### Optimal Chunk Sizes
```json
{
  "chunking": {
    "enabled": true,
    "strategy": {
      "name": "fixed_size",
      "config": {
        "chunk_size": 512,    // Sweet spot for most content
        "chunk_overlap": 51   // ~10% overlap
      }
    }
  }
}
```

#### Content-Aware Chunking
```javascript
// Custom chunking in n8n Code node
function smartChunk(text, maxSize = 512) {
  const sentences = text.split(/[.!?]+/);
  const chunks = [];
  let currentChunk = '';

  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxSize) {
      if (currentChunk) chunks.push(currentChunk.trim());
      currentChunk = sentence;
    } else {
      currentChunk += sentence + '.';
    }
  }

  if (currentChunk) chunks.push(currentChunk.trim());
  return chunks;
}

const text = $input.first().json.content;
const chunks = smartChunk(text);

return chunks.map(chunk => ({ json: { content: chunk } }));
```

### Metadata Best Practices

#### Rich Metadata
```json
{
  "content": "Document content...",
  "metadata": {
    "title": "Descriptive title",
    "summary": "Brief summary for better matching",
    "keywords": ["key", "terms", "for", "search"],
    "category": "product_info",
    "subcategory": "features",
    "language": "en",
    "created_date": "2024-01-01",
    "author": "team@company.com",
    "confidence": 0.95,
    "source_url": "https://...",
    "version": "1.2"
  }
}
```

### Query Optimization

#### Multi-Step Retrieval
```json
// 1. Broad search first
{
  "query": "IONOS AI features",
  "top_k": 10,
  "include_metadata": true
}

// 2. Filter and re-rank (in Code node)
// 3. Final specific query
{
  "query": "specific feature details",
  "top_k": 3,
  "filter": {
    "metadata.category": "features",
    "metadata.confidence": {"$gte": 0.8}
  }
}
```

## 🔄 Workflow Design Patterns

### Error Handling Pattern

#### Comprehensive Error Handling
```json
{
  "nodes": [
    {
      "name": "API Call",
      "type": "n8n-nodes-base.httpRequest"
    },
    {
      "name": "Check Success",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $node['API Call'].json.error === undefined }}",
              "value2": true
            }
          ]
        }
      }
    },
    {
      "name": "Handle Error",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "jsCode": "// Log error and provide fallback\nconsole.error('API Error:', $input.first().json.error);\nreturn [{ json: { error: true, fallback: 'Default response' } }];"
      }
    },
    {
      "name": "Retry Logic",
      "type": "n8n-nodes-base.code"
    }
  ]
}
```

### Retry Pattern with Exponential Backoff

```javascript
// In n8n Code node
async function retryWithBackoff(apiCall, maxRetries = 3) {
  let delay = 1000; // Start with 1 second

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await apiCall();
      return result;
    } catch (error) {
      if (attempt === maxRetries) throw error;

      if (error.status === 429 || error.status >= 500) {
        console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= 2; // Exponential backoff
      } else {
        throw error; // Don't retry client errors (4xx except 429)
      }
    }
  }
}

// Usage example
const result = await retryWithBackoff(async () => {
  // Your API call here
  return await makeAPICall();
});

return [{ json: result }];
```

### Circuit Breaker Pattern

```javascript
// In n8n Code node - Circuit breaker for API reliability
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureThreshold = threshold;
    this.timeout = timeout;
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  }

  async call(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}

// Global circuit breaker instance
const circuitBreaker = new CircuitBreaker();

// Use in your API calls
const result = await circuitBreaker.call(async () => {
  return await fetch('/api/endpoint');
});
```

## 📊 Monitoring and Observability

### Logging Strategy

#### Structured Logging
```javascript
// In n8n Code node
function logEvent(level, event, data = {}) {
  const logEntry = {
    timestamp: new Date().toISOString(),
    level: level,
    event: event,
    workflow_id: '{{ $workflow.id }}',
    execution_id: '{{ $execution.id }}',
    node_name: '{{ $node.name }}',
    ...data
  };

  console.log(JSON.stringify(logEntry));
}

// Usage examples
logEvent('INFO', 'api_call_started', { model: 'llama-3.3-70b' });
logEvent('ERROR', 'api_call_failed', { error: error.message, retries: 3 });
logEvent('SUCCESS', 'document_processed', { chunks: 15, collection_id: 'abc123' });
```

### Performance Monitoring

#### Track API Usage
```javascript
// Monitor token usage and costs
const usage = {
  timestamp: Date.now(),
  model: $input.first().json.model,
  prompt_tokens: response.usage.prompt_tokens,
  completion_tokens: response.usage.completion_tokens,
  total_tokens: response.usage.total_tokens,
  response_time: Date.now() - startTime
};

// Send to monitoring system (e.g., InfluxDB, CloudWatch)
await sendMetrics(usage);
```

### Health Checks

#### API Health Check Workflow
```json
{
  "name": "IONOS Health Check",
  "nodes": [
    {
      "name": "Schedule",
      "type": "n8n-nodes-base.cron",
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "minute": "*/5"  // Every 5 minutes
            }
          ]
        }
      }
    },
    {
      "name": "Check Models API",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "https://openai.inference.de-txl.ionos.com/v1/models",
        "method": "GET"
      }
    },
    {
      "name": "Alert if Down",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{ $node['Check Models API'].statusCode }}",
              "operation": "notEqual",
              "value2": 200
            }
          ]
        }
      }
    }
  ]
}
```

## 🏗️ Deployment Best Practices

### Environment Management

#### Multi-Environment Setup
```bash
# Development
export IONOS_API_KEY="dev_token"
export N8N_BASIC_AUTH_ACTIVE=true
docker-compose -f docker-compose.dev.yml up

# Staging
export IONOS_API_KEY="staging_token"
docker-compose -f docker-compose.staging.yml up

# Production
export IONOS_API_KEY="prod_token"
export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER="admin"
export N8N_BASIC_AUTH_PASSWORD="secure_password"
docker-compose -f docker-compose.prod.yml up
```

### Backup Strategy

#### Export Workflows Regularly
```bash
# Export all workflows
n8n export:workflow --all --output=workflows_backup_$(date +%Y%m%d).json

# Export specific workflow
n8n export:workflow --id=123 --output=critical_workflow.json

# Automated backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
docker exec n8n n8n export:workflow --all --output=/tmp/backup_$DATE.json
docker cp n8n:/tmp/backup_$DATE.json ./backups/
```

### Resource Limits

#### Docker Resource Constraints
```yaml
# docker-compose.yml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '1.0'
        reservations:
          memory: 1G
          cpus: '0.5'
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - IONOS_API_KEY=${IONOS_API_KEY}
```

## 🧪 Testing Best Practices

### Unit Testing Workflows

#### Test Individual Nodes
```javascript
// Test node in isolation
const testInput = {
  json: {
    query: "test query",
    collection_id: "test-collection"
  }
};

// Mock API responses for testing
const mockResponse = {
  documents: [{ content: "test content", score: 0.95 }]
};

// Validate output format
function validateQueryResponse(response) {
  return (
    response.documents &&
    Array.isArray(response.documents) &&
    response.documents.every(doc =>
      doc.content && typeof doc.score === 'number'
    )
  );
}
```

### Integration Testing

#### End-to-End RAG Test
```bash
#!/bin/bash
# rag_integration_test.sh

echo "🧪 Testing IONOS RAG Pipeline"

# 1. Create test collection
COLLECTION_ID=$(curl -s -X POST https://inference.de-txl.ionos.com/collections \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "test-'$(date +%s)'", "engine": {"db_type": "pgvector"}}' | \
  jq -r '.id')

echo "Created collection: $COLLECTION_ID"

# 2. Add test document
curl -s -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"content": "IONOS AI Model Hub provides European AI services with GDPR compliance."}'

# 3. Wait and query
sleep 10
RESULTS=$(curl -s -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/query \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What does IONOS provide?", "top_k": 1}')

# 4. Validate results
if echo "$RESULTS" | grep -q "European AI services"; then
  echo "✅ RAG test passed"
else
  echo "❌ RAG test failed"
  echo "$RESULTS"
fi

# 5. Cleanup
curl -s -X DELETE https://inference.de-txl.ionos.com/collections/$COLLECTION_ID \
  -H "Authorization: Bearer $IONOS_API_KEY"

echo "🧹 Cleanup completed"
```

## 📈 Cost Optimization

### Token Usage Monitoring
```javascript
// Track and optimize token usage
let monthlyTokens = 0;
const TOKEN_LIMIT = 1000000; // 1M tokens per month
const COST_PER_TOKEN = 0.00002; // Example pricing

function trackUsage(usage) {
  monthlyTokens += usage.total_tokens;
  const cost = monthlyTokens * COST_PER_TOKEN;

  if (monthlyTokens > TOKEN_LIMIT * 0.8) {
    console.warn('⚠️ Approaching token limit:', {
      used: monthlyTokens,
      limit: TOKEN_LIMIT,
      cost: cost.toFixed(2)
    });
  }
}
```

### Smart Model Selection
```javascript
// Choose model based on complexity and cost
function selectOptimalModel(taskComplexity, budget) {
  const models = [
    { name: 'meta-llama/Meta-Llama-3.1-8B-Instruct', cost: 0.1, capability: 7 },
    { name: 'meta-llama/Llama-3.3-70B-Instruct', cost: 0.8, capability: 9 },
    { name: 'meta-llama/Meta-Llama-3.1-405B-Instruct-FP8', cost: 2.0, capability: 10 }
  ];

  return models
    .filter(m => m.cost <= budget && m.capability >= taskComplexity)
    .sort((a, b) => a.cost - b.cost)[0]; // Cheapest that meets requirements
}
```

---

## 📋 Checklist: Production-Ready Workflow

- [ ] **Security**
  - [ ] API tokens in environment variables
  - [ ] No hardcoded credentials
  - [ ] Proper access controls

- [ ] **Performance**
  - [ ] Appropriate model selection
  - [ ] Optimized token limits
  - [ ] Response caching implemented
  - [ ] Parallel processing where possible

- [ ] **Reliability**
  - [ ] Error handling for all API calls
  - [ ] Retry logic with exponential backoff
  - [ ] Circuit breaker for external dependencies
  - [ ] Health checks implemented

- [ ] **Monitoring**
  - [ ] Structured logging
  - [ ] Performance metrics collection
  - [ ] Cost tracking
  - [ ] Alert thresholds set

- [ ] **Testing**
  - [ ] Unit tests for complex logic
  - [ ] Integration tests for API calls
  - [ ] End-to-end workflow validation
  - [ ] Load testing completed

- [ ] **Documentation**
  - [ ] Workflow purpose documented
  - [ ] Configuration instructions
  - [ ] Troubleshooting guide
  - [ ] Runbook for operations

---

**Ready to deploy?** Review this checklist and ensure all items are addressed before moving to production!