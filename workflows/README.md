# 🔄 n8n Workflow Examples

This directory contains ready-to-use n8n workflow JSON files for IONOS AI Model Hub integration.

## 📁 Available Workflows

### 1. Basic Chat (`basic-chat.json`)
**Simple chat completion using IONOS AI Model Hub**

- **Nodes**: 2
- **Complexity**: Beginner
- **Models**: Llama 3.3 70B
- **Use Case**: Simple AI chat responses

**Features:**
- OpenAI-compatible API usage
- Environment variable authentication
- Configurable temperature and max tokens

---

### 2. RAG Pipeline (`rag-pipeline.json`)
**Complete Retrieval-Augmented Generation system**

- **Nodes**: 5
- **Complexity**: Intermediate
- **Models**: Llama 3.3 70B
- **Use Case**: Document-based AI responses

**Features:**
- Vector database creation (PgVector)
- Document ingestion and chunking
- Semantic search
- Context-aware AI responses

**Workflow Steps:**
1. Create vector collection
2. Add document content
3. Query collection for relevant context
4. Generate AI response with context

---

### 3. Multi-Model Comparison (`multi-model.json`)
**Compare responses from different AI models**

- **Nodes**: 4
- **Complexity**: Intermediate
- **Models**: Llama 3.3 70B, CodeLlama 13B
- **Use Case**: Model comparison and evaluation

**Features:**
- Parallel model execution
- Response comparison
- Code generation focus

**Workflow Steps:**
1. Send same prompt to multiple models
2. Collect responses in parallel
3. Use AI to compare and analyze differences

---

## 🚀 How to Use

### 1. Import Workflow
1. Download the JSON file
2. Open n8n
3. **Settings** → **Import from File**
4. Select the JSON file
5. Click **Import**

### 2. Configure Environment
Make sure you have the IONOS API key set:
```bash
export IONOS_API_KEY="your_jwt_token_here"
```

### 3. Execute Workflow
1. Click **Manual Trigger** node
2. Click **Execute Workflow**
3. Review results in node outputs

## 🔧 Customization Tips

### Change Models
Replace model IDs in HTTP Request nodes:
```json
{
  "model": "meta-llama/Mistral-Small-24B-Instruct"
}
```

### Modify Parameters
Adjust AI behavior:
```json
{
  "temperature": 0.3,     // More focused (0.0-1.0)
  "max_tokens": 500,      // Longer responses
  "top_p": 0.8           // Nucleus sampling
}
```

### Add Error Handling
Use IF nodes to check for errors:
```json
{
  "conditions": {
    "string": [
      {
        "value1": "{{ $node['IONOS Chat'].json.error }}",
        "operation": "isEmpty"
      }
    ]
  }
}
```

### Use Dynamic Content
Reference previous node outputs:
```json
{
  "content": "{{ $node['Previous Node'].json.result }}"
}
```

## 🔍 Workflow Patterns

### Sequential Processing
```
Trigger → Process A → Process B → Result
```

### Parallel Processing
```
         → Process A →
Trigger             → Combine → Result
         → Process B →
```

### Conditional Logic
```
Trigger → Check → IF True → Action A
                → IF False → Action B
```

## 📊 Performance Tips

1. **Use appropriate models**:
   - Small tasks: Mistral Small, Llama 8B
   - Complex reasoning: Llama 405B
   - Code: CodeLlama

2. **Optimize token usage**:
   - Set reasonable `max_tokens`
   - Use lower `temperature` for focused responses
   - Trim unnecessary context

3. **Handle rate limits**:
   - Add wait nodes between requests
   - Implement exponential backoff
   - Monitor API usage

## 🛠️ Troubleshooting

### Common Issues

**401 Unauthorized**
- Check `IONOS_API_KEY` environment variable
- Verify token permissions in IONOS dashboard

**Model Not Found**
- Check model ID spelling
- Verify model is available in your region

**Rate Limit Exceeded**
- Add delay between requests
- Reduce concurrent executions
- Contact IONOS for higher limits

### Debug Tips

1. **Use Code nodes** to log variables:
```javascript
console.log('API Response:', $node['IONOS Chat'].json);
return $input.all();
```

2. **Check node outputs** in the right panel
3. **Enable workflow logs** in settings
4. **Test with simple examples** first

## 🔄 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-18 | Initial workflows |
| | | Basic chat, RAG pipeline, multi-model |

---

**Need help?** Check the main [documentation](../docs/) or ask in the [n8n community](https://community.n8n.io)!