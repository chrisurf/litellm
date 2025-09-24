# Custom LLM Configuration Guide

This guide covers how to configure and use local LLMs and custom deployments with LiteLLM, giving you full control over your data and models while maintaining the same unified API interface.

## 📋 Table of Contents

- [Overview](#overview)
- [Local LLM Options](#local-llm-options)
- [Configuration Examples](#configuration-examples)
- [Proxy Server Setup](#proxy-server-setup)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## 🏠 Overview

LiteLLM supports multiple local LLM deployment options, allowing you to:
- **Run models on your own infrastructure** - Full data control and privacy
- **Use the same API interface** - Consistent experience across providers
- **Route requests intelligently** - Mix local and cloud providers
- **Maintain cost control** - No per-token charges for local models

## 🔧 Local LLM Options

### 1. Ollama

**Description:** Local LLM runner with easy model management  
**Best for:** Development, experimentation, and lightweight deployments

#### Direct Usage
```bash
# Start LiteLLM with Ollama model
litellm --model ollama/llama2
```

#### Python Usage
```python
import litellm

response = litellm.completion(
    model="ollama/llama2",
    messages=[{"role": "user", "content": "Hello!"}],
    api_base="http://localhost:11434"  # Default Ollama port
)
```

#### Configuration
```yaml
model_list:
  - model_name: local-llama2
    litellm_params:
      model: ollama/llama2
      api_base: http://localhost:11434
```

---

### 2. VLLM

**Description:** High-performance inference server for large language models  
**Best for:** Production deployments requiring high throughput

#### Python Usage
```python
import litellm

response = litellm.completion(
    model="hosted_vllm/facebook/opt-125m",
    messages=[{"role": "user", "content": "What's the weather like?"}],
    api_base="https://your-vllm-server.com",
    temperature=0.2,
    max_tokens=80
)
```

#### Configuration
```yaml
model_list:
  - model_name: vllm-opt-125m
    litellm_params:
      model: hosted_vllm/facebook/opt-125m
      api_base: https://your-vllm-server.com
      
  - model_name: vllm-llama-7b
    litellm_params:
      model: hosted_vllm/meta-llama/Llama-2-7b-chat-hf
      api_base: https://your-vllm-server.com
```

---

### 3. LM Studio

**Description:** Desktop application for running LLMs locally with a user-friendly interface  
**Best for:** Local development and testing

#### Python Usage
```python
import litellm

response = litellm.completion(
    model="lm_studio/llama-3-8b-instruct",
    messages=[{
        "role": "user",
        "content": "What's the weather like in Boston today in Fahrenheit?"
    }],
    api_base="http://localhost:1234"  # Default LM Studio port
)
```

#### Configuration
```yaml
model_list:
  - model_name: lm-studio-llama3
    litellm_params:
      model: lm_studio/llama-3-8b-instruct
      api_base: http://localhost:1234
```

---

### 4. OpenAI-Compatible Servers

**Description:** Any server that implements the OpenAI API format  
**Best for:** Custom deployments and specialized setups

#### Direct Usage
```bash
# Start LiteLLM with custom OpenAI-compatible server
litellm --model openai/your-custom-model --api_base http://localhost:8000
```

#### Python Usage
```python
import litellm

response = litellm.completion(
    model="openai/custom-model",
    messages=[{"role": "user", "content": "Hello!"}],
    api_base="http://localhost:8000",
    api_key="your-custom-key"  # If authentication is required
)
```

#### Configuration
```yaml
model_list:
  - model_name: custom-gpt
    litellm_params:
      model: openai/custom-gpt-model
      api_base: http://localhost:8000
      api_key: os.environ/CUSTOM_API_KEY
```

---

### 5. Text Generation WebUI (Oobabooga)

**Description:** Popular web interface for running various language models  
**Best for:** Experimentation with different models and parameters

#### Configuration
```yaml
model_list:
  - model_name: oobabooga-model
    litellm_params:
      model: openai/text-davinci-003  # Dummy model name
      api_base: http://localhost:5000/v1
      api_key: dummy  # Oobabooga doesn't require real API key
```

---

### 6. llamafile

**Description:** Single-file executable that runs LLMs  
**Best for:** Simple deployments and edge computing

#### Configuration
```yaml
model_list:
  - model_name: llamafile-model
    litellm_params:
      model: openai/gpt-3.5-turbo  # Dummy model name
      api_base: http://localhost:8080
```

## ⚙️ Configuration Examples

### Mixed Local and Cloud Setup

```yaml
model_list:
  # Local models
  - model_name: local-llama2
    litellm_params:
      model: ollama/llama2
      api_base: http://localhost:11434
      
  - model_name: local-vllm
    litellm_params:
      model: hosted_vllm/meta-llama/Llama-2-7b-chat-hf
      api_base: http://localhost:8000
  
  # Cloud models
  - model_name: gpt-4
    litellm_params:
      model: openai/gpt-4
      api_key: os.environ/OPENAI_API_KEY
      
  - model_name: claude-3
    litellm_params:
      model: anthropic/claude-3-sonnet-20240229
      api_key: os.environ/ANTHROPIC_API_KEY

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  database_url: os.environ/DATABASE_URL
```

### Development Environment

```yaml
model_list:
  # Fast local model for development
  - model_name: dev-model
    litellm_params:
      model: ollama/codellama:7b
      api_base: http://localhost:11434
      
  # Production-like local model
  - model_name: staging-model
    litellm_params:
      model: lm_studio/llama-3-70b-instruct
      api_base: http://localhost:1234
```

### High-Performance Setup

```yaml
model_list:
  # Multiple VLLM instances for load balancing
  - model_name: vllm-cluster-1
    litellm_params:
      model: hosted_vllm/meta-llama/Llama-2-70b-chat-hf
      api_base: http://vllm-node-1:8000
      
  - model_name: vllm-cluster-2
    litellm_params:
      model: hosted_vllm/meta-llama/Llama-2-70b-chat-hf
      api_base: http://vllm-node-2:8000
      
  - model_name: vllm-cluster-3
    litellm_params:
      model: hosted_vllm/meta-llama/Llama-2-70b-chat-hf
      api_base: http://vllm-node-3:8000
```

## 🐳 Proxy Server Setup

### Docker Compose with Local Models

```yaml
version: "3.9"

services:
  # LiteLLM Proxy
  litellm:
    image: ghcr.io/berriai/litellm:main-stable
    ports:
      - "4000:4000"
    volumes:
      - ./litellm-config.yaml:/app/config.yaml
    environment:
      - LITELLM_MASTER_KEY=${LITELLM_MASTER_KEY}
    command: ["--config", "/app/config.yaml", "--port", "4000"]
    depends_on:
      - ollama
  
  # Ollama for local LLM serving
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    environment:
      - OLLAMA_HOST=0.0.0.0
    # Pull models on startup
    entrypoint: ["/bin/bash", "-c", "ollama serve & sleep 5 && ollama pull llama2 && wait"]

volumes:
  ollama_data:
```

### Environment Variables for Local Setup

```bash
# .env file
LITELLM_MASTER_KEY=your-secure-master-key

# Local model endpoints
OLLAMA_API_BASE=http://ollama:11434
VLLM_API_BASE=http://localhost:8000
LM_STUDIO_API_BASE=http://localhost:1234

# Optional: Custom API keys for authenticated local services
CUSTOM_API_KEY=your-custom-key
```

## 🎯 Best Practices

### 1. Model Selection
- **Development**: Use smaller, faster models (7B parameters)
- **Production**: Choose models based on quality vs. speed requirements
- **Fallback**: Configure cloud models as fallbacks for local failures

### 2. Resource Management
```yaml
# Example resource-aware configuration
model_list:
  - model_name: small-local
    litellm_params:
      model: ollama/llama2:7b
      api_base: http://localhost:11434
    # Route lightweight requests here
    
  - model_name: large-local  
    litellm_params:
      model: hosted_vllm/meta-llama/Llama-2-70b-chat-hf
      api_base: http://localhost:8000
    # Route complex requests here
```

### 3. Health Checks
```yaml
general_settings:
  health_check: true
  health_check_interval: 60  # seconds
```

### 4. Monitoring and Logging
```yaml
general_settings:
  set_verbose: true
  success_callback: ["langfuse"]  # Track usage
  failure_callback: ["langfuse"]  # Track failures
```

## 🛠️ Troubleshooting

### Common Issues

#### 1. Connection Refused
```bash
# Check if local server is running
curl http://localhost:11434/api/tags  # Ollama
curl http://localhost:8000/v1/models  # VLLM
curl http://localhost:1234/v1/models  # LM Studio
```

#### 2. Model Not Found
```bash
# List available models
curl http://localhost:11434/api/tags  # Ollama models
```

#### 3. Memory Issues
- Monitor GPU/CPU usage
- Adjust model size or batch size
- Consider model quantization

#### 4. Slow Response Times
- Check hardware resources
- Optimize model parameters
- Consider model caching

### Debug Mode

Enable detailed logging:

```yaml
general_settings:
  set_verbose: true
  debug: true
```

### Testing Local Setup

```bash
# Test local model via LiteLLM proxy
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer your-master-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "local-llama2",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

## 📊 Performance Considerations

### Hardware Requirements

| Model Size | RAM Required | GPU Memory | Recommended Setup |
|------------|-------------|------------|-------------------|
| 7B params  | 16GB       | 8GB        | RTX 3080/4070     |
| 13B params | 32GB       | 16GB       | RTX 4080/A4000    |
| 30B params | 64GB       | 24GB       | RTX 4090/A5000    |
| 70B params | 128GB      | 48GB       | A100/H100         |

### Optimization Tips

1. **Use quantized models** when possible (4-bit, 8-bit)
2. **Enable GPU acceleration** for compatible models
3. **Configure appropriate batch sizes** for your hardware
4. **Use model caching** to avoid reloading
5. **Monitor resource usage** and adjust accordingly

## 🔗 Additional Resources

- [Ollama Documentation](https://ollama.ai/docs)
- [VLLM Documentation](https://docs.vllm.ai/)
- [LM Studio Documentation](https://lmstudio.ai/docs)
- [LiteLLM Provider Documentation](https://docs.litellm.ai/docs/providers)

---

**💡 Pro Tip:** Start with Ollama for experimentation, then move to VLLM for production workloads requiring high throughput.