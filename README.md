# LiteLLM Docker Repository

A complete Docker setup for running [LiteLLM](https://docs.litellm.ai/) proxy server with PostgreSQL database, web UI, and support for multiple LLM providers including OpenAI, Azure OpenAI, and more.

## ✨ Features

- 🐘 **PostgreSQL Database** - For persistent data, user management, and spend tracking
- 🌐 **Web UI** - Admin interface for managing keys, users, and monitoring usage
- 🔑 **Virtual Keys** - Create and manage API keys with budgets and permissions
- 📊 **Usage Analytics** - Track spending and usage across models and users
- 🔒 **Authentication** - Secure access with master keys and user management
- 🚀 **Multiple LLM Support** - OpenAI, Azure OpenAI, and more providers

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

Edit `.env` file with your actual API keys and database credentials:

```bash
# OpenAI Configuration (Primary)
OPENAI_API_KEY=your-openai-api-key-here

# PostgreSQL Database Configuration
POSTGRES_DB=litellm
POSTGRES_USER=litellm
POSTGRES_PASSWORD=your-secure-postgres-password

# LiteLLM Authentication
LITELLM_MASTER_KEY=your-secure-master-key

# Web UI Access
UI_USERNAME=admin
UI_PASSWORD=your-secure-ui-password

# Optional: Azure OpenAI Configuration
# AZURE_API_KEY=your-azure-api-key-here
# AZURE_API_BASE=https://your-resource-name.openai.azure.com/
```

### 3. Launch LiteLLM with PostgreSQL

```bash
docker compose up -d
```

This will start:
- 🐘 **PostgreSQL database** on port 5432
- 🚀 **LiteLLM proxy** on port 4000
- 🌐 **Web UI** at http://localhost:4000

### 4. Access the Web UI

Open your browser and navigate to: **http://localhost:4000**

Login with:
- **Username**: `admin` (or your `UI_USERNAME`)
- **Password**: Your `UI_PASSWORD` from `.env`

### 5. Test the Setup

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

If you prefer not to use Docker Compose (you'll need to set up PostgreSQL separately):

```bash
docker run \
  -v $(pwd)/litellm-config.yaml:/app/config.yaml \
  -e OPENAI_API_KEY=your-openai-api-key \
  -e DATABASE_URL=postgresql://user:password@host:5432/litellm \
  -e LITELLM_MASTER_KEY=your-master-key \
  -e UI_USERNAME=admin \
  -e UI_PASSWORD=your-ui-password \
  -p 4000:4000 \
  ghcr.io/berriai/litellm:main-stable \
  --config /app/config.yaml --port 4000 --detailed_debug
```

## ⚙️ Configuration Options

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENAI_API_KEY` | OpenAI API key | Yes (for OpenAI) |
| `POSTGRES_PASSWORD` | PostgreSQL database password | Yes |
| `LITELLM_MASTER_KEY` | Master key for LiteLLM authentication | Yes |
| `UI_PASSWORD` | Password for web UI access | Yes |
| `POSTGRES_DB` | PostgreSQL database name | Optional (default: litellm) |
| `POSTGRES_USER` | PostgreSQL username | Optional (default: litellm) |
| `UI_USERNAME` | Username for web UI access | Optional (default: admin) |
| `AZURE_API_KEY` | Azure OpenAI API key | Optional |
| `AZURE_API_BASE` | Azure OpenAI endpoint URL | Optional |

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

### Web UI Management

Access the web UI at http://localhost:4000 to:
- 👥 **Manage Users** - Create and manage user accounts
- 🔑 **Virtual Keys** - Generate API keys with budgets and rate limits
- 📊 **Usage Analytics** - Monitor spending and usage patterns
- ⚙️ **Model Configuration** - Add and configure LLM providers
- 🏢 **Team Management** - Organize users into teams with budgets

### Virtual Keys API

Create virtual keys programmatically:

```bash
# Create a virtual key with budget
curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer your-master-key" \
  -H "Content-Type: application/json" \
  -d '{
    "models": ["gpt-4", "gpt-4o"],
    "max_budget": 10.0,
    "duration": "30d"
  }'
```

### 📖 REST API Reference

For complete programmatic management of users, teams, and keys, see the comprehensive REST API documentation:

**➡️ [LiteLLM REST API Reference](./LITELLM_REST_API.md)**

The API reference includes:
- 👥 **User Management** - Create, update, delete, and list users
- 🏢 **Team Management** - Manage teams and team memberships  
- 🔑 **Key Management** - Generate, update, and delete API keys
- 📊 **Usage Tracking** - Monitor spending and usage analytics
- 🔐 **Authentication** - Secure access with proper permissions

### Database Features

The PostgreSQL database stores:
- 📊 **Usage Logs** - All API requests and costs in `LiteLLM_SpendLogs`
- 🔑 **Virtual Keys** - API keys with permissions and budgets
- 👥 **Users & Teams** - User management and organization
- 💰 **Budgets** - Spend tracking and limits
- ⚙️ **Model Configs** - When `STORE_MODEL_IN_DB=True`

### Enable Authentication

Authentication is automatically enabled with the master key. Users can:
- Use the master key for admin access
- Generate virtual keys for limited access
- Access the web UI with UI credentials

### Add Redis Caching

Uncomment the Redis service in `docker-compose.yml` and add to your config:

```yaml
general_settings:
  redis_host: redis
  redis_port: 6379
```

### External Database

To use an external PostgreSQL database instead of the Docker container:

```bash
# In your .env file
DATABASE_URL=postgresql://user:password@external-host:5432/litellm
```

Then remove the `postgres` service from `docker-compose.yml`.

## 📊 Monitoring and Logs

### View Logs

```bash
# View LiteLLM logs
docker compose logs -f litellm

# View PostgreSQL logs
docker compose logs -f postgres

# View logs for a specific time period
docker compose logs --since 1h litellm
```

### Database Access

Connect to PostgreSQL directly:

```bash
# Access PostgreSQL container
docker compose exec postgres psql -U litellm -d litellm

# Or connect from host (if port 5432 is available)
psql postgresql://litellm:your-password@localhost:5432/litellm
```

### Health Checks

- **LiteLLM Health**: http://localhost:4000/health
- **Database Status**: Check with `docker compose ps`

## 🛠️ Troubleshooting

### Common Issues

1. **Port already in use**: Change port mappings in `docker-compose.yml`
2. **Environment variables not loaded**: Ensure `.env` file is in the same directory
3. **Database connection failed**: Check PostgreSQL container status and credentials
4. **Web UI not accessible**: Verify `UI_USERNAME` and `UI_PASSWORD` are set
5. **OpenAI API key invalid**: Verify your OpenAI API key is correct and has sufficient credits
6. **Model not found**: Ensure you're using supported OpenAI model names (gpt-4, gpt-4o, gpt-3.5-turbo, etc.)
7. **Permission denied**: Ensure proper file permissions for config files

### Database Issues

```bash
# Reset PostgreSQL data (WARNING: This deletes all data)
docker compose down -v
docker compose up -d

# Check database connectivity
docker compose exec postgres pg_isready -U litellm
```

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