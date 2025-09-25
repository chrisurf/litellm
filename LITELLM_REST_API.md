# LiteLLM REST API Reference

A comprehensive REST API reference for managing LiteLLM proxy server configuration, including user management, team management, and key management.

## 📋 Table of Contents

- [Authentication](#authentication)
- [User Management](#user-management)
- [Team Management](#team-management)
- [Key Management](#key-management)
- [Model Management](#model-management)
- [Customer Management](#customer-management)
- [Vector Store Management](#vector-store-management)
- [MCP Management](#mcp-management)
- [OpenAI-Compatible API](#openai-compatible-api)
- [Administrative Features](#administrative-features)
- [Common Response Formats](#common-response-formats)
- [Error Handling](#error-handling)

## 🔐 Authentication

All management endpoints require authentication using the master key or an admin-level API key in the `Authorization` header.

```bash
Authorization: Bearer sk-your-master-key-here
```

### Required Headers

```bash
Content-Type: application/json
Authorization: Bearer sk-your-master-key-here
```

---

## 👥 User Management

### Create New User

**Endpoint:** `POST /user/new`  
**Description:** Create a new internal user with budget and model access controls.

```bash
curl -X POST 'http://localhost:4000/user/new' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "user_email": "alice@company.com",
    "user_role": "internal_user",
    "user_alias": "Alice Smith",
    "teams": ["team-123"],
    "max_budget": 100.0,
    "budget_duration": "30d",
    "models": ["gpt-4", "gpt-3.5-turbo"],
    "tpm_limit": 1000,
    "rpm_limit": 100,
    "metadata": {
      "department": "engineering",
      "employee_id": "EMP001"
    },
    "send_invite_email": true,
    "auto_create_key": true
  }'
```

**Parameters:**
- `user_email` (string, optional): User's email address
- `user_role` (string, optional): Role - "proxy_admin", "internal_user", "team", "customer"
- `user_alias` (string, optional): Descriptive name for the user
- `teams` (array, optional): List of team IDs the user belongs to
- `max_budget` (float, optional): Maximum budget for the user
- `budget_duration` (string, optional): Budget reset period - "30s", "30m", "30h", "30d", "1mo"
- `models` (array, optional): Models the user can access
- `tpm_limit` (integer, optional): Tokens per minute limit
- `rpm_limit` (integer, optional): Requests per minute limit
- `metadata` (object, optional): Additional user information
- `send_invite_email` (boolean, optional): Send invitation email
- `auto_create_key` (boolean, optional): Automatically create API key

**Response:**
```json
{
  "key": "sk-1234ewknldferwedojwojw",
  "expires": "2024-01-25T10:30:00Z",
  "user_id": "user-456",
  "user_email": "alice@company.com",
  "user_role": "internal_user",
  "max_budget": 100.0,
  "teams": ["team-123"]
}
```

### Update Existing User

**Endpoint:** `POST /user/update`  
**Description:** Update an existing user's settings and permissions.

```bash
curl -X POST 'http://localhost:4000/user/update' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "user_id": "user-456",
    "user_email": "alice.updated@company.com",
    "user_role": "proxy_admin",
    "max_budget": 200.0,
    "models": ["gpt-4", "claude-3"],
    "metadata": {
      "department": "management",
      "promotion_date": "2024-01-15"
    }
  }'
```

**Parameters:**
- `user_id` (string, required): ID of the user to update
- Other parameters same as `/user/new`

### Delete Users

**Endpoint:** `POST /user/delete`  
**Description:** Delete users and their associated API keys.

```bash
curl -X POST 'http://localhost:4000/user/delete' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "user_ids": ["user-456", "user-789"]
  }'
```

**Parameters:**
- `user_ids` (array, required): List of user IDs to delete

### List Users

**Endpoint:** `GET /user/list`  
**Description:** List users with filtering and pagination.

```bash
# List all users with pagination
curl -X GET 'http://localhost:4000/user/list?page=1&page_size=10' \
  -H 'Authorization: Bearer sk-1234'

# Filter by role
curl -X GET 'http://localhost:4000/user/list?user_role=internal_user' \
  -H 'Authorization: Bearer sk-1234'

# Filter by team
curl -X GET 'http://localhost:4000/user/list?team_id=team-123' \
  -H 'Authorization: Bearer sk-1234'
```

**Query Parameters:**
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Items per page (default: 10)
- `user_role` (string, optional): Filter by user role
- `team_id` (string, optional): Filter by team ID

**Response:**
```json
{
  "users": [
    {
      "user_id": "user-456",
      "user_email": "alice@company.com",
      "user_role": "internal_user",
      "spend": 45.50,
      "max_budget": 100.0,
      "key_count": 3
    }
  ],
  "total": 25,
  "page": 1,
  "page_size": 10,
  "total_pages": 3
}
```

### Get User Information

**Endpoint:** `GET /user/info`  
**Description:** Get detailed information about a specific user.

```bash
curl -X GET 'http://localhost:4000/user/info?user_id=user-456' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "user_id": "user-456",
  "user_email": "alice@company.com",
  "user_role": "internal_user",
  "spend": 45.50,
  "max_budget": 100.0,
  "teams": ["team-123"],
  "keys": [
    {
      "token": "sk-...",
      "spend": 45.50,
      "models": ["gpt-4"]
    }
  ]
}
```

---

## 🏢 Team Management

### Create New Team

**Endpoint:** `POST /team/new`  
**Description:** Create a new team with budget controls and member management.

```bash
curl -X POST 'http://localhost:4000/team/new' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_alias": "Engineering Team",
    "members_with_roles": [
      {
        "role": "admin",
        "user_id": "user-123"
      },
      {
        "role": "user",
        "user_email": "alice@company.com"
      }
    ],
    "max_budget": 500.0,
    "budget_duration": "30d",
    "tpm_limit": 10000,
    "rpm_limit": 1000,
    "models": ["gpt-4", "gpt-3.5-turbo"],
    "metadata": {
      "department": "engineering",
      "cost_center": "eng-001"
    },
    "team_member_budget": 50.0,
    "team_member_rpm_limit": 100,
    "team_member_tpm_limit": 1000
  }'
```

**Parameters:**
- `team_alias` (string, optional): Human-readable team name
- `team_id` (string, optional): Custom team ID (auto-generated if not provided)
- `members_with_roles` (array, optional): Team members with their roles
- `max_budget` (float, optional): Maximum budget for the team
- `budget_duration` (string, optional): Budget reset period
- `tpm_limit` (integer, optional): Team-wide tokens per minute limit
- `rpm_limit` (integer, optional): Team-wide requests per minute limit
- `models` (array, optional): Models the team can access
- `metadata` (object, optional): Additional team information
- `team_member_budget` (float, optional): Individual member budget limit
- `team_member_rpm_limit` (integer, optional): Individual member RPM limit
- `team_member_tpm_limit` (integer, optional): Individual member TPM limit

**Response:**
```json
{
  "team_id": "team-456",
  "team_alias": "Engineering Team",
  "members_with_roles": [
    {
      "role": "admin",
      "user_id": "user-123"
    }
  ],
  "max_budget": 500.0,
  "budget_duration": "30d",
  "tpm_limit": 10000,
  "rpm_limit": 1000,
  "models": ["gpt-4", "gpt-3.5-turbo"],
  "spend": 0.0
}
```

### Update Team Settings

**Endpoint:** `POST /team/update`  
**Description:** Update team configuration and settings.

```bash
curl -X POST 'http://localhost:4000/team/update' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_id": "team-456",
    "team_alias": "Updated Engineering Team",
    "max_budget": 750.0,
    "budget_duration": "60d",
    "tpm_limit": 15000,
    "rpm_limit": 1500,
    "models": ["gpt-4", "gpt-3.5-turbo", "claude-3"],
    "metadata": {
      "department": "engineering",
      "cost_center": "eng-002",
      "updated": true
    }
  }'
```

### Add Team Members

**Endpoint:** `POST /team/member_add`  
**Description:** Add users to a team with specific roles and budget limits.

```bash
# Add single member
curl -X POST 'http://localhost:4000/team/member_add' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_id": "team-456",
    "member": {
      "role": "user",
      "user_email": "bob@company.com"
    },
    "max_budget_in_team": 100.0
  }'

# Add multiple members
curl -X POST 'http://localhost:4000/team/member_add' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_id": "team-456",
    "member": [
      {
        "role": "user",
        "user_id": "user-789"
      },
      {
        "role": "admin",
        "user_email": "charlie@company.com"
      }
    ]
  }'
```

**Parameters:**
- `team_id` (string, required): Team ID to add members to
- `member` (object/array, required): Member(s) to add with roles
- `max_budget_in_team` (float, optional): Individual budget limit for team member

**Response:**
```json
{
  "team_id": "team-456",
  "team_alias": "Engineering Team",
  "members_with_roles": [
    {
      "role": "admin",
      "user_id": "user-123"
    },
    {
      "role": "user",
      "user_id": "user-789",
      "user_email": "bob@company.com"
    }
  ],
  "updated_users": [
    {
      "user_id": "user-789",
      "user_email": "bob@company.com",
      "teams": ["team-456"]
    }
  ]
}
```

###me Team Members

**Endpoint:** `POST /team/member_delete`  
**Description:** Remove users from a team.

```bash
# Remove member by user_id
curl -X POST 'http://localhost:4000/team/member_delete' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_id": "team-456",
    "user_id": "user-789"
  }'

# Remove member by user_email
curl -X POST 'http://localhost:4000/team/member_delete' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_id": "team-456",
    "user_email": "bob@company.com"
  }'
```

**Response:**
```json
{
  "message": "Member removed successfully",
  "team_id": "team-456",
  "removed_user": "user-789"
}
```

---

## 🔑 Key Management

### Generate API Key

**Endpoint:** `POST /key/generate`  
**Description:** Generate a new API key with specific permissions and limits.

```bash
curl -X POST 'http://localhost:4000/key/generate' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "models": ["gpt-4", "gpt-3.5-turbo"],
    "aliases": {"gpt4": "gpt-4"},
    "duration": "24h",
    "key_alias": "my-project-key",
    "team_id": "team-123",
    "user_id": "user-456",
    "max_budget": 50.0,
    "tpm_limit": 1000,
    "rpm_limit": 100,
    "metadata": {
      "project": "chatbot-v2",
      "environment": "production"
    },
    "guardrails": ["aporia-pre-call"],
    "tags": ["production", "chatbot"]
  }'
```

**Parameters:**
- `models` (array, optional): Models the key can access
- `aliases` (object, optional): Model alias mappings
- `duration` (string, optional): Key expiration - "30s", "30m", "30h", "30d"
- `key_alias` (string, optional): Human-readable key name
- `team_id` (string, optional): Associate key with team
- `user_id` (string, optional): Associate key with user
- `max_budget` (float, optional): Maximum spending limit
- `tpm_limit` (integer, optional): Tokens per minute limit
- `rpm_limit` (integer, optional): Requests per minute limit
- `metadata` (object, optional): Additional key information
- `guardrails` (array, optional): Active guardrails for the key
- `tags` (array, optional): Tags for tracking and routing

**Response:**
```json
{
  "key": "sk-1234ewknldferwedojwojw",
  "expires": "2024-01-25T10:30:00Z",
  "user_id": "user-456",
  "team_id": "team-123",
  "key_alias": "my-project-key",
  "spend": 0.0,
  "max_budget": 50.0
}
```

### Update API Key

**Endpoint:** `POST /key/update`  
**Description:** Update an existing API key's settings and permissions.

```bash
curl -X POST 'http://localhost:4000/key/update' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "key": "sk-1234ewknldferwedojwojw",
    "models": ["gpt-4", "claude-3"],
    "max_budget": 75.0,
    "tpm_limit": 2000,
    "rpm_limit": 200,
    "guardrails": ["aporia-pre-call", "aporia-post-call"],
    "metadata": {
      "project": "chatbot-v3",
      "updated": true
    }
  }'
```

### Delete API Keys

**Endpoint:** `POST /key/delete`  
**Description:** Delete one or more API keys.

```bash
curl -X POST 'http://localhost:4000/key/delete' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "keys": ["sk-1234ewknldferwedojwojw", "sk-anotherkeyhere"]
  }'
```

### Get Key Information

**Endpoint:** `GET /key/info`  
**Description:** Get detailed information about a specific API key.

```bash
curl -X GET 'http://localhost:4000/key/info?key=sk-1234ewknldferwedojwojw' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "key": "sk-1234ewknldferwedojwojw",
  "info": {
    "user_id": "user-456",
    "team_id": "team-123",
    "spend": 25.75,
    "max_budget": 75.0,
    "models": ["gpt-4", "claude-3"],
    "tpm_limit": 2000,
    "rpm_limit": 200,
    "expires": "2024-01-25T10:30:00Z",
    "metadata": {
      "project": "chatbot-v3",
      "guardrails": ["aporia-pre-call", "aporia-post-call"]
    },
    "tags": ["production", "chatbot"]
  }
}
```

### Generate Service Account Key

**Endpoint:** `POST /key/service-account/generate`  
**Description:** Generate a service account key that belongs to a team rather than a user.

```bash
curl -X POST 'http://localhost:4000/key/service-account/generate' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "team_id": "team-456",
    "models": ["gpt-4", "gpt-3.5-turbo"],
    "max_budget": 100.0,
    "key_alias": "service-account-key",
    "metadata": {
      "service": "automated-reports",
      "type": "service_account"
    }
  }'
```

---

## 🤖 Model Management

### Add New Model

**Endpoint:** `POST /model/new`  
**Description:** Add a new model configuration to the proxy.

```bash
curl -X POST 'http://localhost:4000/model/new' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model_name": "custom-gpt-4",
    "litellm_params": {
      "model": "openai/gpt-4",
      "api_key": "os.environ/OPENAI_API_KEY",
      "api_base": "https://api.openai.com/v1"
    },
    "model_info": {
      "description": "Custom GPT-4 configuration",
      "max_tokens": 8192,
      "cost_per_token": 0.00003
    }
  }'
```

**Parameters:**
- `model_name` (string, required): Unique name for the model
- `litellm_params` (object, required): Model configuration parameters
- `model_info` (object, optional): Additional model metadata

### Update Model Configuration

**Endpoint:** `POST /model/update`  
**Description:** Update an existing model's configuration.

```bash
curl -X POST 'http://localhost:4000/model/update' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model_name": "custom-gpt-4",
    "litellm_params": {
      "model": "openai/gpt-4-turbo",
      "api_key": "os.environ/OPENAI_API_KEY"
    },
    "model_info": {
      "description": "Updated to GPT-4 Turbo",
      "max_tokens": 128000
    }
  }'
```

### Delete Model

**Endpoint:** `POST /model/delete`  
**Description:** Remove a model from the proxy configuration.

```bash
curl -X POST 'http://localhost:4000/model/delete' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model_name": "custom-gpt-4"
  }'
```

### Get Model Information

**Endpoint:** `GET /model/info`  
**Description:** Get detailed information about a specific model.

```bash
curl -X GET 'http://localhost:4000/model/info?model_name=custom-gpt-4' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "model_name": "custom-gpt-4",
  "litellm_params": {
    "model": "openai/gpt-4-turbo",
    "api_key": "os.environ/OPENAI_API_KEY"
  },
  "model_info": {
    "description": "Updated to GPT-4 Turbo",
    "max_tokens": 128000,
    "cost_per_token": 0.00003
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-20T14:45:00Z"
}
```

---

## 👥 Customer Management

### List Customers

**Endpoint:** `GET /customer/list`  
**Description:** List all customers/end-users in the system.

```bash
# List all customers with pagination
curl -X GET 'http://localhost:4000/customer/list?page=1&page_size=10' \
  -H 'Authorization: Bearer sk-1234'

# Filter customers
curl -X GET 'http://localhost:4000/customer/list?customer_email=user@company.com' \
  -H 'Authorization: Bearer sk-1234'
```

**Query Parameters:**
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Items per page (default: 10)
- `customer_email` (string, optional): Filter by customer email
- `customer_id` (string, optional): Filter by customer ID

**Response:**
```json
{
  "customers": [
    {
      "customer_id": "customer-123",
      "customer_email": "user@company.com",
      "spend": 45.75,
      "total_requests": 1250,
      "created_at": "2024-01-10T09:00:00Z",
      "last_active": "2024-01-25T16:30:00Z"
    }
  ],
  "total": 156,
  "page": 1,
  "page_size": 10,
  "total_pages": 16
}
```

### List End Users (Alternative Endpoint)

**Endpoint:** `GET /end_user/list`  
**Description:** Alternative endpoint for listing end-users with similar functionality.

```bash
curl -X GET 'http://localhost:4000/end_user/list?page=1&page_size=20' \
  -H 'Authorization: Bearer sk-1234'
```

---

## 🗂️ Vector Store Management

**Note:** This is an Enterprise feature for vector store management.

### Create Vector Store

**Endpoint:** `POST /vector_store/new`  
**Description:** Create a new vector store for document embeddings and retrieval.

```bash
curl -X POST 'http://localhost:4000/vector_store/new' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "company-docs",
    "description": "Company documentation embeddings",
    "embedding_model": "text-embedding-ada-002",
    "metadata": {
      "department": "engineering",
      "document_type": "technical"
    }
  }'
```

**Parameters:**
- `name` (string, required): Unique name for the vector store
- `description` (string, optional): Description of the vector store
- `embedding_model` (string, optional): Model to use for embeddings
- `metadata` (object, optional): Additional metadata

**Response:**
```json
{
  "vector_store_id": "vs-abc123",
  "name": "company-docs",
  "description": "Company documentation embeddings",
  "embedding_model": "text-embedding-ada-002",
  "status": "active",
  "created_at": "2024-01-25T10:30:00Z"
}
```

### Delete Vector Store

**Endpoint:** `POST /vector_store/delete`  
**Description:** Delete a vector store and all associated data.

```bash
curl -X POST 'http://localhost:4000/vector_store/delete' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "vector_store_id": "vs-abc123"
  }'
```

### List Vector Stores

**Endpoint:** `GET /vector_store/list`  
**Description:** List all vector stores with filtering options.

```bash
curl -X GET 'http://localhost:4000/vector_store/list?status=active' \
  -H 'Authorization: Bearer sk-1234'
```

**Query Parameters:**
- `status` (string, optional): Filter by status (active, inactive)
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page

**Response:**
```json
{
  "vector_stores": [
    {
      "vector_store_id": "vs-abc123",
      "name": "company-docs",
      "description": "Company documentation embeddings",
      "document_count": 1500,
      "size_mb": 250.5,
      "status": "active",
      "created_at": "2024-01-25T10:30:00Z"
    }
  ],
  "total": 5,
  "page": 1,
  "page_size": 10
}
```

---

## 🔧 MCP Management

**Description:** Model Context Protocol (MCP) management for tool integration.

### Get Available MCP Tools

**Endpoint:** `GET /v1/mcp/tools`  
**Description:** Get list of available MCP tools and their capabilities.

```bash
curl -X GET 'http://localhost:4000/v1/mcp/tools' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "tools": [
    {
      "name": "web_search",
      "description": "Search the web for information",
      "parameters": {
        "query": {
          "type": "string",
          "description": "Search query"
        },
        "max_results": {
          "type": "integer",
          "description": "Maximum number of results"
        }
      }
    },
    {
      "name": "code_executor",
      "description": "Execute code in various languages",
      "parameters": {
        "code": {
          "type": "string",
          "description": "Code to execute"
        },
        "language": {
          "type": "string",
          "description": "Programming language"
        }
      }
    }
  ]
}
```

---

## 🔌 OpenAI-Compatible API

LiteLLM provides full OpenAI API compatibility. All endpoints use the same format as OpenAI's API.

### Core LLM APIs

#### Chat Completions
```bash
curl -X POST 'http://localhost:4000/v1/chat/completions' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "Hello!"}],
    "temperature": 0.7,
    "max_tokens": 100
  }'
```

#### Text Completions
```bash
curl -X POST 'http://localhost:4000/v1/completions' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "gpt-3.5-turbo-instruct",
    "prompt": "Once upon a time",
    "max_tokens": 50
  }'
```

#### Embeddings
```bash
curl -X POST 'http://localhost:4000/v1/embeddings' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "text-embedding-ada-002",
    "input": "The quick brown fox jumps over the lazy dog"
  }'
```

#### Content Moderation
```bash
curl -X POST 'http://localhost:4000/v1/moderations' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "input": "I want to hurt someone"
  }'
```

### Advanced Features

#### Image Generation
```bash
curl -X POST 'http://localhost:4000/v1/images/generations' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "prompt": "A futuristic city skyline",
    "n": 1,
    "size": "1024x1024"
  }'
```

#### Speech-to-Text
```bash
curl -X POST 'http://localhost:4000/v1/audio/transcriptions' \
  -H 'Authorization: Bearer sk-1234' \
  -F 'file=@audio.mp3' \
  -F 'model=whisper-1'
```

#### Text-to-Speech
```bash
curl -X POST 'http://localhost:4000/v1/audio/speech' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "tts-1",
    "input": "Hello, this is a test of text to speech.",
    "voice": "alloy"
  }'
```

#### Fine-tuning Management
```bash
# Create fine-tuning job
curl -X POST 'http://localhost:4000/v1/fine_tuning/jobs' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "training_file": "file-abc123",
    "model": "gpt-3.5-turbo"
  }'

# List fine-tuning jobs
curl -X GET 'http://localhost:4000/v1/fine_tuning/jobs' \
  -H 'Authorization: Bearer sk-1234'
```

#### Assistants API
```bash
# Create assistant
curl -X POST 'http://localhost:4000/v1/assistants' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "gpt-4",
    "name": "Customer Support Assistant",
    "instructions": "You are a helpful customer support assistant."
  }'

# Create thread
curl -X POST 'http://localhost:4000/v1/threads' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

### Batch Processing
```bash
# Create batch job
curl -X POST 'http://localhost:4000/v1/batches' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "input_file_id": "file-abc123",
    "endpoint": "/v1/chat/completions",
    "completion_window": "24h"
  }'
```

---

## ⚙️ Administrative Features

### Configuration Management

#### Health Check
```bash
curl -X GET 'http://localhost:4000/health' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "status": "healthy",
  "database": "connected",
  "models": "loaded",
  "uptime": "2d 14h 23m",
  "version": "1.0.0"
}
```

#### Prometheus Metrics
```bash
curl -X GET 'http://localhost:4000/metrics' \
  -H 'Authorization: Bearer sk-1234'
```

#### Configuration Updates
```bash
# Update proxy configuration
curl -X POST 'http://localhost:4000/config/update' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "general_settings": {
      "max_budget": 1000.0,
      "budget_duration": "30d"
    }
  }'
```

### Global Operations

#### Global Spend Reset
```bash
curl -X POST 'http://localhost:4000/global/spend/reset' \
  -H 'Authorization: Bearer sk-1234' \
  -H 'Content-Type: application/json' \
  -d '{
    "confirm": true,
    "reset_type": "monthly"
  }'
```

#### Memory Usage Monitoring
```bash
curl -X GET 'http://localhost:4000/global/memory/usage' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "memory_usage_mb": 2048,
  "memory_limit_mb": 4096,
  "cpu_usage_percent": 45.2,
  "active_connections": 127,
  "cache_size_mb": 512
}
```

### Audit and Monitoring

#### Audit Logs
```bash
# Get audit logs with filtering
curl -X GET 'http://localhost:4000/audit/logs?action=create&table_name=user&limit=50' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "audit_logs": [
    {
      "id": "audit-123",
      "table_name": "user",
      "object_id": "user-456",
      "action": "create",
      "changed_by": "admin-user",
      "changed_at": "2024-01-25T10:30:00Z",
      "before_value": null,
      "after_value": {
        "user_email": "new@company.com",
        "user_role": "internal_user"
      }
    }
  ],
  "total": 1,
  "page": 1,
  "page_size": 50
}
```

**Tracked Actions:**
- `create` - Entity creation
- `update` - Entity modification  
- `delete` - Entity deletion
- `regenerate` - Key regeneration

**Tracked Entities:**
- Users
- Teams
- API Keys
- Models
- Vector Stores

### Access Control Configuration

#### Endpoint Security Settings

**Environment Variables:**
```bash
# Disable admin/management endpoints
DISABLE_ADMIN_ENDPOINTS=true

# Disable LLM API endpoints
DISABLE_LLM_API_ENDPOINTS=true

# Enable only specific endpoint categories
ENABLE_ENDPOINTS=user_management,key_management
```

#### Role-Based Access Control
```bash
# Check user permissions
curl -X GET 'http://localhost:4000/user/permissions?user_id=user-456' \
  -H 'Authorization: Bearer sk-1234'
```

**Response:**
```json
{
  "user_id": "user-456",
  "role": "internal_user",
  "permissions": [
    "key:generate",
    "key:info",
    "model:list",
    "chat:completions"
  ],
  "restricted_endpoints": [
    "user:delete",
    "team:create",
    "model:delete"
  ]
}
```

---

## 📊 Common Response Formats

### Success Response

```json
{
  "status": "success",
  "data": {
    // Response data here
  }
}
```

### Pagination Response

```json
{
  "items": [],
  "total": 100,
  "page": 1,
  "page_size": 10,
  "total_pages": 10
}
```

---

## ❌ Error Handling

### Error Response Format

```json
{
  "error": {
    "message": "Error description",
    "type": "error_type",
    "code": 400
  }
}
```

### Common HTTP Status Codes

- `200` - Success
- `400` - Bad Request (Invalid parameters)
- `401` - Unauthorized (Invalid API key)
- `403` - Forbidden (Insufficient permissions)
- `404` - Not Found (Resource doesn't exist)
- `429` - Rate Limited
- `500` - Internal Server Error

### Common Error Types

- `authentication_error` - Invalid API key or authentication
- `permission_denied` - Insufficient permissions for operation
- `validation_error` - Invalid request parameters
- `resource_not_found` - Requested resource doesn't exist
- `rate_limit_exceeded` - API rate limit exceeded
- `budget_exceeded` - Budget limit reached

---

## 🔒 Security & Best Practices

### Authentication
- Always use HTTPS in production
- Keep master keys secure and rotate regularly
- Use service account keys for automated systems
- Implement proper key management practices

### Rate Limiting
- Set appropriate TPM/RPM limits based on usage patterns
- Monitor usage to prevent abuse
- Use soft budgets for early warnings

### Budget Control
- Set reasonable budget limits for users and teams
- Monitor spending regularly
- Use budget alerts for proactive management

### Access Control
- Follow principle of least privilege
- Use teams to organize access permissions
- Regularly audit user permissions and team memberships

---

## 🚀 Getting Started

1. **Set up your master key** in your LiteLLM configuration
2. **Create teams** for organizing users and permissions
3. **Add users** and assign them to appropriate teams
4. **Generate API keys** with proper limits and permissions
5. **Monitor usage** through the web UI and API endpoints

For more detailed information, visit the [LiteLLM Documentation](https://docs.litellm.ai/).