# 🔧 Troubleshooting IONOS AI Model Hub + n8n

Common issues and solutions when working with IONOS AI Model Hub in n8n workflows.

## 🚨 Authentication Issues

### 401 Unauthorized

**Symptoms:**
```json
{
  "httpStatus": 401,
  "messages": [{"errorCode": "paas-auth-1", "message": "Unauthorized, wrong or no api key provided"}]
}
```

**Solutions:**

1. **Check Environment Variable**
```bash
# Verify token is set
docker exec n8n printenv | grep IONOS
echo $IONOS_API_KEY
```

2. **Verify Token Format**
```bash
# Token should start with 'eyJ'
echo $IONOS_API_KEY | head -c 10
# Should output: eyJ0eXAiOi
```

3. **Check Token Expiration**
```bash
# Decode token (install jq first)
echo $IONOS_API_KEY | cut -d'.' -f2 | base64 -d | jq '.exp'
# Compare with current timestamp: date +%s
```

4. **Verify Token Permissions**
- Login to IONOS Cloud Dashboard
- Go to **Access Management** → **API Tokens**
- Check token has `ACCESS_AND_MANAGE_AI_MODEL_HUB` permission

5. **Regenerate Token**
```bash
# If token is expired or corrupted, create new one
# Update environment and restart n8n
export IONOS_API_KEY="new_token_here"
docker restart n8n
```

### Wrong Header Format in n8n

**Problem:** Authorization header not properly formatted

**Solution:**
```json
{
  "headerParameters": {
    "parameters": [
      {
        "name": "Authorization",
        "value": "Bearer {{ $env.IONOS_API_KEY }}"
      }
    ]
  }
}
```

**Not:**
```json
{
  "name": "Authorization",
  "value": "{{ $env.IONOS_API_KEY }}"  // ❌ Missing "Bearer "
}
```

## 🌐 API Endpoint Issues

### 404 Not Found

**Common Wrong URLs:**
```
❌ https://api.ionos.com/modelhub/v1/
❌ https://api.ionos.com/inference/v1/
❌ https://inference.ionos.com/
```

**Correct URLs:**
```
✅ https://inference.de-txl.ionos.com/
✅ https://openai.inference.de-txl.ionos.com/v1/
```

### Regional Endpoints

**Issue:** API not available in your region

**Check Available Regions:**
```bash
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/models
```

**Solution:** Currently only `de-txl` (Germany) region is available

## 🤖 Model Issues

### Model Not Found

**Error:**
```json
{
  "error": {
    "message": "The model 'gpt-3.5-turbo' does not exist",
    "type": "invalid_request_error"
  }
}
```

**Solution:** Use IONOS model names
```json
{
  "model": "meta-llama/Llama-3.3-70B-Instruct"  // ✅ Correct
}
```

**Available Models:**
```bash
# List all available models
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://openai.inference.de-txl.ionos.com/v1/models | jq -r '.data[].id'
```

### Model Loading Issues

**Symptoms:** Long response times or timeouts

**Solutions:**
1. **Use smaller models** for testing:
   - `meta-llama/Meta-Llama-3.1-8B-Instruct` (faster)
   - `mistralai/Mistral-Nemo-Instruct-2407` (efficient)

2. **Increase timeout** in n8n HTTP Request node:
```json
{
  "timeout": 120000  // 2 minutes
}
```

## 📚 Collection Issues

### Collection Not Found

**Error:**
```json
{
  "error": "Collection not found"
}
```

**Solutions:**

1. **Check Collection ID**
```bash
# List all collections
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/collections
```

2. **Wait for Creation**
```javascript
// In n8n Code node - wait for collection creation
await new Promise(resolve => setTimeout(resolve, 5000));
return $input.all();
```

3. **Use Collection Name Instead**
```bash
# Find by name
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     "https://inference.de-txl.ionos.com/collections?name=my-collection"
```

### Collection Creation Failures

**Common Issues:**

1. **Duplicate Names**
```json
{
  "name": "my-collection-{{ $now }}"  // ✅ Add timestamp
}
```

2. **Invalid Configuration**
```json
{
  "engine": {
    "db_type": "pgvector"  // ✅ Use "pgvector" not "postgres"
  }
}
```

3. **Missing Chunking Strategy**
```json
{
  "chunking": {
    "enabled": true,
    "strategy": {
      "name": "fixed_size",     // ✅ Required when enabled
      "config": {
        "chunk_size": 512,
        "chunk_overlap": 50
      }
    }
  }
}
```

## 📄 Document Processing Issues

### Document Not Added

**Symptoms:** Documents appear to add but don't show in queries

**Solutions:**

1. **Wait for Processing**
```bash
# Check document status
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents
```

2. **Add Processing Delay**
```javascript
// In n8n Code node
await new Promise(resolve => setTimeout(resolve, 10000)); // 10 second wait
return $input.all();
```

3. **Check Content Length**
```json
{
  "content": "Your content must be at least 10 characters long..."
}
```

### Chunking Issues

**Problem:** Documents too large or poorly chunked

**Solutions:**

1. **Optimize Chunk Size**
```json
{
  "chunking": {
    "strategy": {
      "config": {
        "chunk_size": 256,    // Smaller for better precision
        "chunk_overlap": 25   // 10% overlap recommended
      }
    }
  }
}
```

2. **Pre-process Long Documents**
```javascript
// In n8n Code node - split long content
const content = $input.first().json.content;
const maxLength = 1000;

if (content.length > maxLength) {
  const chunks = [];
  for (let i = 0; i < content.length; i += maxLength) {
    chunks.push({
      content: content.slice(i, i + maxLength),
      metadata: { chunk: Math.floor(i / maxLength) + 1 }
    });
  }
  return chunks;
}

return [{ content, metadata: {} }];
```

## 🔍 Query Issues

### No Results Found

**Symptoms:** Queries return empty results despite having documents

**Solutions:**

1. **Check Query Similarity**
```bash
# Test with exact content match first
curl -X POST https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/query \
  -H "Authorization: Bearer $IONOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "exact phrase from your document",
    "top_k": 1
  }'
```

2. **Increase top_k**
```json
{
  "top_k": 10  // Get more results
}
```

3. **Simplify Query**
```json
{
  "query": "simple keywords"  // Instead of complex questions
}
```

4. **Check Document Status**
```bash
# Ensure documents are processed
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/collections/$COLLECTION_ID/documents
```

### Poor Query Results

**Issue:** Results not relevant to query

**Solutions:**

1. **Use Better Embedding Model**
```json
{
  "embedding_model": "BAAI/bge-large-en-v1.5"  // Better for English
}
```

2. **Add Metadata Filters**
```json
{
  "query": "your question",
  "filter": {
    "metadata.type": "relevant_type",
    "metadata.category": "specific_category"
  }
}
```

3. **Improve Document Metadata**
```json
{
  "content": "...",
  "metadata": {
    "title": "Document Title",
    "summary": "Brief summary",
    "keywords": ["key", "terms"],
    "category": "specific_category"
  }
}
```

## 🌊 Rate Limiting

### 429 Too Many Requests

**Error:**
```json
{
  "error": {
    "message": "Rate limit exceeded",
    "type": "rate_limit_error"
  }
}
```

**Solutions:**

1. **Add Delays in n8n**
```json
// Wait node configuration
{
  "amount": 2,
  "unit": "seconds"
}
```

2. **Implement Exponential Backoff**
```javascript
// In n8n Code node
const maxRetries = 3;
let delay = 1000; // Start with 1 second

for (let retry = 0; retry < maxRetries; retry++) {
  try {
    // Your API call here
    break;
  } catch (error) {
    if (error.status === 429 && retry < maxRetries - 1) {
      await new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // Double the delay
    } else {
      throw error;
    }
  }
}
```

3. **Reduce Concurrent Requests**
- Process documents sequentially instead of parallel
- Use smaller batch sizes

## 💾 n8n Specific Issues

### Environment Variables Not Working

**Problem:** `{{ $env.IONOS_API_KEY }}` returns empty

**Solutions:**

1. **Check Docker Environment**
```bash
docker exec n8n printenv | grep IONOS
```

2. **Restart n8n Container**
```bash
docker restart n8n
```

3. **Use Alternative Methods**
```javascript
// In n8n Code node
const apiKey = process.env.IONOS_API_KEY;
$input.first().json.apiKey = apiKey;
return $input.all();
```

### Workflow Import Issues

**Problem:** JSON import fails or nodes missing

**Solutions:**

1. **Check n8n Version Compatibility**
```bash
docker exec n8n n8n --version
```

2. **Update Node Type Versions**
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2  // Use latest version
}
```

3. **Import Step by Step**
- Import basic workflow first
- Add nodes manually if needed
- Copy/paste configurations

### Expression Errors

**Problem:** Node expressions not resolving correctly

**Common Issues:**
```javascript
// ❌ Wrong syntax
{{ $node['Node Name'].json.field }}

// ✅ Correct syntax
{{ $node["Node Name"].json["field"] }}
{{ $('Node Name').first().json.field }}
```

**Debugging Expressions:**
```javascript
// In Code node - log all available data
console.log('Input data:', JSON.stringify($input.all(), null, 2));
console.log('Previous node:', JSON.stringify($node["Previous Node"].json, null, 2));
return $input.all();
```

## 🔄 Workflow Optimization

### Slow Performance

**Solutions:**

1. **Optimize Model Selection**
```json
{
  "model": "meta-llama/Meta-Llama-3.1-8B-Instruct"  // Faster than 70B
}
```

2. **Reduce Token Limits**
```json
{
  "max_tokens": 100  // Instead of 1000+
}
```

3. **Use Parallel Processing**
```javascript
// Split work across multiple HTTP Request nodes
// Connect them all to the same trigger
```

4. **Cache Responses**
```javascript
// Store results to avoid repeat API calls
const cache = {};
const cacheKey = JSON.stringify($input.first().json);

if (cache[cacheKey]) {
  return [{ json: cache[cacheKey] }];
}
// ... make API call and store in cache
```

## 📞 Getting Help

### IONOS Support
- 📧 support@ionos.com
- 🌐 [IONOS Cloud Status](https://www.ionos-status.com/)
- 📖 [Official Documentation](https://docs.ionos.com/cloud/ai/ai-model-hub)

### n8n Community
- 💬 [n8n Community Forum](https://community.n8n.io/)
- 📖 [n8n Documentation](https://docs.n8n.io/)
- 🐛 [GitHub Issues](https://github.com/n8n-io/n8n/issues)

### Debug Information to Include
When asking for help, include:

```bash
# System info
docker exec n8n n8n --version
docker exec n8n node --version

# Error details
echo "Error: [paste full error message]"
echo "Workflow: [paste relevant workflow JSON]"
echo "Expected: [what you expected to happen]"
echo "Actual: [what actually happened]"

# API test
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://inference.de-txl.ionos.com/models
```

---

**Still having issues?** Create a minimal example that reproduces the problem and ask for help in the [n8n community](https://community.n8n.io/) with the "ionos" tag.