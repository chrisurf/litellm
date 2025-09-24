# LiteLLM REST API Reference

A comprehensive REST API reference for managing LiteLLM proxy server configuration, including user management, team management, and key management.

## 📋 Table of Contents

- [Authentication](#authentication)
- [User Management](#user-management)
- [Team Management](#team-management)
- [Key Management](#key-management)
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