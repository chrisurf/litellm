# LiteLLM Docker Repository

A simple and ready-to-use Docker setup for running [LiteLLM](https://docs.litellm.ai/) proxy server with support for multiple LLM providers including OpenAI, Azure OpenAI, and more.

## 📁 Repository Structure

```
litellm-docker-repo/
├── litellm-config.yaml    # LiteLLM configuration file
├── docker-compose.yml     # Docker Compose setup
├── .env.example          # Environment variables template
└── README.md             # This file
```

## 🚀 Quick Start

### 1. Clone and Setup

```bash
git clone git@github.com:chrisurf/litellm.git
cd litellm
```

### 2. Configure Environment Variables

Copy the example environment file and edit it with your API credentials:

```bash
cp .env.example .env
```

Edit `.env` file with your actual API keys:

```bash
# OpenAI Configuration (Primary)
OPENAI_API_KEY=your-openai-api-key-here

# Optional: Azure OpenAI Configuration
# AZURE_API_KEY=your-azure-api-key-here
# AZURE_API_BASE=https://your-resource-name.openai.azure.com/

# Optional: LiteLLM Master Key for authentication
# LITELLM_MASTER_KEY=your-master-key-here
```

### 3. Launch LiteLLM Proxy

```bash
docker compose up -d
```

The LiteLLM proxy will be available at: **http://localhost:4000**

### 4. Test the Setup

```bash
# Health check
curl http://localhost:4000/health

# List available models
curl http://localhost:4000/v1/models

# Test chat completion with GPT-4
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'

# Test chat completion with GPT-4o
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## 🐳 Manual Docker Run

If you prefer not to use Docker Compose:

```bash
docker run \
  -v $(pwd)/litellm-config.yaml:/app/config.yaml \
  -e OPENAI_API_KEY=your-openai-api-key \
  -p 4000:4000 \
  ghcr.io/berriai/litellm:main-stable \
  --config /app/config.yaml --port 4000 --detailed_debug
```

## ⚙️ Configuration Options

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENAI_API_KEY` | OpenAI API key | Yes (for OpenAI) |
| `AZURE_API_KEY` | Azure OpenAI API key | Optional |
| `AZURE_API_BASE` | Azure OpenAI endpoint URL | Optional |
| `LITELLM_MASTER_KEY` | Master key for LiteLLM authentication | Optional |

### Adding More Models

Edit `litellm-config.yaml` to add additional models:

```yaml
model_list:
  # OpenAI GPT-4
  - model_name: gpt-4
    litellm_params:
      model: openai/gpt-4
      api_key: os.environ/OPENAI_API_KEY
  
  # OpenAI GPT-4o
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY
  
  # OpenAI GPT-3.5 Turbo
  - model_name: gpt-3.5-turbo
    litellm_params:
      model: openai/gpt-3.5-turbo
      api_key: os.environ/OPENAI_API_KEY
  
  # Azure GPT-4o (if using Azure)
  - model_name: azure-gpt-4o
    litellm_params:
      model: azure/your-gpt-4o-deployment
      api_base: os.environ/AZURE_API_BASE
      api_key: os.environ/AZURE_API_KEY
      api_version: "2025-01-01-preview"
```

## 🔧 Advanced Features

### Enable Authentication

Add a master key to your `.env` file:

```bash
LITELLM_MASTER_KEY=your-secure-master-key
```

Then update `litellm-config.yaml`:

```yaml
general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
```

### Add Redis Caching

Uncomment the Redis service in `docker-compose.yml` and add to your config:

```yaml
general_settings:
  redis_host: redis
  redis_port: 6379
```

### Database Integration

For persistent logging and analytics, add a database URL:

```bash
DATABASE_URL=postgresql://user:password@host:port/database
```

## 📊 Monitoring and Logs

### View Logs

```bash
# View real-time logs
docker compose logs -f litellm

# View logs for a specific time period
docker compose logs --since 1h litellm
```

### Health Check

The container includes a built-in health check accessible at:
```
http://localhost:4000/health
```

## 🛠️ Troubleshooting

### Common Issues

1. **Port already in use**: Change the port mapping in `docker-compose.yml`
2. **Environment variables not loaded**: Ensure `.env` file is in the same directory
3. **OpenAI API key invalid**: Verify your OpenAI API key is correct and has sufficient credits
4. **Model not found**: Ensure you're using supported OpenAI model names (gpt-4, gpt-4o, gpt-3.5-turbo, etc.)
5. **Permission denied**: Ensure proper file permissions for the config file

### Debug Mode

For detailed debugging, modify the command in `docker-compose.yml`:

```yaml
command: ["--config", "/app/config.yaml", "--port", "4000", "--detailed_debug"]
```

## 📚 Additional Resources

- [LiteLLM Documentation](https://docs.litellm.ai/)
- [LiteLLM Proxy Documentation](https://docs.litellm.ai/docs/proxy/deploy)
- [Supported LLM Providers](https://docs.litellm.ai/docs/providers)

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.