# All settings

```yaml
environment_variables: {}

model_list:
  - model_name: string
    litellm_params: {}
    model_info:
      id: string
      mode: embedding
      input_cost_per_token: 0
      output_cost_per_token: 0
      max_tokens: 2048
      base_model: gpt-4-1106-preview
      additionalProp1: {}
litellm_settings:
  # Logging/Callback settings
  success_callback: ["langfuse"]  # list of success callbacks
  failure_callback: ["sentry"]  # list of failure callbacks
  callbacks: ["otel"]  # list of callbacks - runs on success and failure
  service_callbacks: ["datadog", "prometheus"]  # logs redis, postgres failures on datadog, prometheus
  turn_off_message_logging: boolean  # prevent the messages and responses from being logged to on your callbacks, but request metadata will still be logged.
  redact_user_api_key_info: boolean  # Redact information about the user api key (hashed token, user_id, team id, etc.), from logs. Currently supported for Langfuse, OpenTelemetry, Logfire, ArizeAI logging.
  langfuse_default_tags: ["cache_hit", "cache_key", "proxy_base_url", "user_api_key_alias", "user_api_key_user_id", "user_api_key_user_email", "user_api_key_team_alias", "semantic-similarity", "proxy_base_url"] # default tags for Langfuse Logging
  
  # Networking settings
  request_timeout: 10 # (int) llm requesttimeout in seconds. Raise Timeout error if call takes longer than 10s. Sets litellm.request_timeout 
  force_ipv4: boolean # If true, litellm will force ipv4 for all LLM requests. Some users have seen httpx ConnectionError when using ipv6 + Anthropic API
  
  set_verbose: boolean # sets litellm.set_verbose=True to view verbose debug logs. DO NOT LEAVE THIS ON IN PRODUCTION
  json_logs: boolean # if true, logs will be in json format

  # Fallbacks, reliability
  default_fallbacks: ["claude-opus"] # set default_fallbacks, in case a specific model group is misconfigured / bad.
  content_policy_fallbacks: [{"gpt-3.5-turbo-small": ["claude-opus"]}] # fallbacks for ContentPolicyErrors
  context_window_fallbacks: [{"gpt-3.5-turbo-small": ["gpt-3.5-turbo-large", "claude-opus"]}] # fallbacks for ContextWindowExceededErrors

  # MCP Aliases - Map aliases to MCP server names for easier tool access
  mcp_aliases: { "github": "github_mcp_server", "zapier": "zapier_mcp_server", "deepwiki": "deepwiki_mcp_server" } # Maps friendly aliases to MCP server names. Only the first alias for each server is used

  # Caching settings
  cache: true 
  cache_params:        # set cache params for redis
    type: redis        # type of cache to initialize

    # Optional - Redis Settings
    host: "localhost"  # The host address for the Redis cache. Required if type is "redis".
    port: 6379  # The port number for the Redis cache. Required if type is "redis".
    password: "your_password"  # The password for the Redis cache. Required if type is "redis".
    namespace: "litellm.caching.caching" # namespace for redis cache
  
    # Optional - Redis Cluster Settings
    redis_startup_nodes: [{"host": "127.0.0.1", "port": "7001"}] 

    # Optional - Redis Sentinel Settings
    service_name: "mymaster"
    sentinel_nodes: [["localhost", 26379]]

    # Optional - GCP IAM Authentication for Redis
    gcp_service_account: "projects/-/serviceAccounts/your-sa@project.iam.gserviceaccount.com"  # GCP service account for IAM authentication
    gcp_ssl_ca_certs: "./server-ca.pem"  # Path to SSL CA certificate file for GCP Memorystore Redis
    ssl: true  # Enable SSL for secure connections
    ssl_cert_reqs: null  # Set to null for self-signed certificates
    ssl_check_hostname: false  # Set to false for self-signed certificates

    # Optional - Qdrant Semantic Cache Settings
    qdrant_semantic_cache_embedding_model: openai-embedding # the model should be defined on the model_list
    qdrant_collection_name: test_collection
    qdrant_quantization_config: binary
    similarity_threshold: 0.8   # similarity threshold for semantic cache

    # Optional - S3 Cache Settings
    s3_bucket_name: cache-bucket-litellm   # AWS Bucket Name for S3
    s3_region_name: us-west-2              # AWS Region Name for S3
    s3_aws_access_key_id: os.environ/AWS_ACCESS_KEY_ID  # us os.environ/<variable name> to pass environment variables. This is AWS Access Key ID for S3
    s3_aws_secret_access_key: os.environ/AWS_SECRET_ACCESS_KEY  # AWS Secret Access Key for S3
    s3_endpoint_url: https://s3.amazonaws.com  # [OPTIONAL] S3 endpoint URL, if you want to use Backblaze/cloudflare s3 bucket

    # Common Cache settings
    # Optional - Supported call types for caching
    supported_call_types: ["acompletion", "atext_completion", "aembedding", "atranscription"]
                          # /chat/completions, /completions, /embeddings, /audio/transcriptions
    mode: default_off # if default_off, you need to opt in to caching on a per call basis
    ttl: 600 # ttl for caching
    disable_copilot_system_to_assistant: False  # If false (default), converts all 'system' role messages to 'assistant' for GitHub Copilot compatibility. Set to true to disable this behavior.

callback_settings:
  otel:
    message_logging: boolean  # OTEL logging callback specific settings
general_settings:
  completion_model: string
  disable_spend_logs: boolean  # turn off writing each transaction to the db
  disable_master_key_return: boolean  # turn off returning master key on UI (checked on '/user/info' endpoint)
  disable_retry_on_max_parallel_request_limit_error: boolean  # turn off retries when max parallel request limit is reached
  disable_reset_budget: boolean  # turn off reset budget scheduled task
  disable_adding_master_key_hash_to_db: boolean  # turn off storing master key hash in db, for spend tracking
  enable_jwt_auth: boolean  # allow proxy admin to auth in via jwt tokens with 'litellm_proxy_admin' in claims
  enforce_user_param: boolean  # requires all openai endpoint requests to have a 'user' param
  allowed_routes: ["route1", "route2"]  # list of allowed proxy API routes - a user can access. (currently JWT-Auth only)
  key_management_system: google_kms  # either google_kms or azure_kms
  master_key: string
  maximum_spend_logs_retention_period: 30d # The maximum time to retain spend logs before deletion.
  maximum_spend_logs_retention_interval: 1d # interval in which the spend log cleanup task should run in.

  # Database Settings
  database_url: string
  database_connection_pool_limit: 0  # default 100
  database_connection_timeout: 0  # default 60s
  allow_requests_on_db_unavailable: boolean  # if true, will allow requests that can not connect to the DB to verify Virtual Key to still work 

  custom_auth: string
  max_parallel_requests: 0  # the max parallel requests allowed per deployment 
  global_max_parallel_requests: 0  # the max parallel requests allowed on the proxy all up 
  infer_model_from_keys: true
  background_health_checks: true
  health_check_interval: 300
  alerting: ["slack", "email"]
  alerting_threshold: 0
  use_client_credentials_pass_through_routes: boolean  # use client credentials for all pass through routes like "/vertex-ai", /bedrock/. When this is True Virtual Key auth will not be applied on these endpoints
```

### litellm_settings - Reference

| Name | Type | Description |
|------|------|-------------|
| success_callback | array of strings | List of success callbacks. [Doc Proxy logging callbacks](logging), [Doc Metrics](prometheus) |
| failure_callback | array of strings | List of failure callbacks [Doc Proxy logging callbacks](logging), [Doc Metrics](prometheus) |
| callbacks | array of strings | List of callbacks - runs on success and failure [Doc Proxy logging callbacks](logging), [Doc Metrics](prometheus) |
| service_callbacks | array of strings | System health monitoring - Logs redis, postgres failures on specified services (e.g. datadog, prometheus) [Doc Metrics](prometheus) |
| turn_off_message_logging | boolean | If true, prevents messages and responses from being logged to callbacks, but request metadata will still be logged [Proxy Logging](logging) |
| modify_params | boolean | If true, allows modifying the parameters of the request before it is sent to the LLM provider |
| enable_preview_features | boolean | If true, enables preview features - e.g. Azure O1 Models with streaming support.|
| redact_user_api_key_info | boolean | If true, redacts information about the user api key from logs [Proxy Logging](logging#redacting-userapikeyinfo) |
| mcp_aliases | object | Maps friendly aliases to MCP server names for easier tool access. Only the first alias for each server is used. [MCP Aliases](../mcp#mcp-aliases) |
| langfuse_default_tags | array of strings | Default tags for Langfuse Logging. Use this if you want to control which LiteLLM-specific fields are logged as tags by the LiteLLM proxy. By default LiteLLM Proxy logs no LiteLLM-specific fields as tags. [Further docs](./logging#litellm-specific-tags-on-langfuse---cache_hit-cache_key) |
| set_verbose | boolean | If true, sets litellm.set_verbose=True to view verbose debug logs. DO NOT LEAVE THIS ON IN PRODUCTION |
| json_logs | boolean | If true, logs will be in json format. If you need to store the logs as JSON, just set the `litellm.json_logs = True`. We currently just log the raw POST request from litellm as a JSON [Further docs](./debugging) |
| default_fallbacks | array of strings | List of fallback models to use if a specific model group is misconfigured / bad. [Further docs](./reliability#default-fallbacks) |
| request_timeout | integer | The timeout for requests in seconds. If not set, the default value is `6000 seconds`. [For reference OpenAI Python SDK defaults to `600 seconds`.](https://github.com/openai/openai-python/blob/main/src/openai/_constants.py) |
| force_ipv4 | boolean | If true, litellm will force ipv4 for all LLM requests. Some users have seen httpx ConnectionError when using ipv6 + Anthropic API |
| content_policy_fallbacks | array of objects | Fallbacks to use when a ContentPolicyViolationError is encountered. [Further docs](./reliability#content-policy-fallbacks) |
| context_window_fallbacks | array of objects | Fallbacks to use when a ContextWindowExceededError is encountered. [Further docs](./reliability#context-window-fallbacks) |
| cache | boolean | If true, enables caching. [Further docs](./caching) |
| cache_params | object | Parameters for the cache. [Further docs](./caching#supported-cache_params-on-proxy-configyaml) |
| disable_end_user_cost_tracking | boolean | If true, turns off end user cost tracking on prometheus metrics + litellm spend logs table on proxy. |
| disable_end_user_cost_tracking_prometheus_only | boolean | If true, turns off end user cost tracking on prometheus metrics only. |
| key_generation_settings | object | Restricts who can generate keys. [Further docs](./virtual_keys.md#restricting-key-generation) |
| disable_add_transform_inline_image_block | boolean | For Fireworks AI models - if true, turns off the auto-add of `#transform=inline` to the url of the image_url, if the model is not a vision model. |
| disable_hf_tokenizer_download | boolean | If true, it defaults to using the openai tokenizer for all models (including huggingface models). |
| enable_json_schema_validation | boolean | If true, enables json schema validation for all requests. |
| disable_copilot_system_to_assistant | boolean | If false (default), converts all 'system' role messages to 'assistant' for GitHub Copilot compatibility. Set to true to disable this behavior. Useful for tools (like Claude Code) that send system messages, which Copilot does not support. |

### general_settings - Reference

| Name | Type | Description |
|------|------|-------------|
| completion_model | string | The default model to use for completions when `model` is not specified in the request |
| disable_spend_logs | boolean | If true, turns off writing each transaction to the database |
| disable_spend_updates | boolean | If true, turns off all spend updates to the DB. Including key/user/team spend updates. |
| disable_master_key_return | boolean | If true, turns off returning master key on UI. (checked on '/user/info' endpoint) |
| disable_retry_on_max_parallel_request_limit_error | boolean | If true, turns off retries when max parallel request limit is reached |
| disable_reset_budget | boolean | If true, turns off reset budget scheduled task |
| disable_adding_master_key_hash_to_db | boolean | If true, turns off storing master key hash in db |
| enable_jwt_auth | boolean | allow proxy admin to auth in via jwt tokens with 'litellm_proxy_admin' in claims. [Doc on JWT Tokens](token_auth) |
| enforce_user_param | boolean | If true, requires all OpenAI endpoint requests to have a 'user' param. [Doc on call hooks](call_hooks)|
| allowed_routes | array of strings | List of allowed proxy API routes a user can access [Doc on controlling allowed routes](enterprise#control-available-public-private-routes)|
| key_management_system | string | Specifies the key management system. [Doc Secret Managers](../secret) |
| master_key | string | The master key for the proxy [Set up Virtual Keys](virtual_keys) |
| database_url | string | The URL for the database connection [Set up Virtual Keys](virtual_keys) |
| database_connection_pool_limit | integer | The limit for database connection pool [Setting DB Connection Pool limit](#configure-db-pool-limits--connection-timeouts) |
| database_connection_timeout | integer | The timeout for database connections in seconds [Setting DB Connection Pool limit, timeout](#configure-db-pool-limits--connection-timeouts) |
| allow_requests_on_db_unavailable | boolean | If true, allows requests to succeed even if DB is unreachable. **Only use this if running LiteLLM in your VPC** This will allow requests to work even when LiteLLM cannot connect to the DB to verify a Virtual Key [Doc on graceful db unavailability](prod#5-if-running-litellm-on-vpc-gracefully-handle-db-unavailability) |
| custom_auth | string | Write your own custom authentication logic [Doc Custom Auth](virtual_keys#custom-auth) |
| max_parallel_requests | integer | The max parallel requests allowed per deployment |
| global_max_parallel_requests | integer | The max parallel requests allowed on the proxy overall |
| infer_model_from_keys | boolean | If true, infers the model from the provided keys |
| background_health_checks | boolean | If true, enables background health checks. [Doc on health checks](health) |
| health_check_interval | integer | The interval for health checks in seconds [Doc on health checks](health) |
| alerting | array of strings | List of alerting methods [Doc on Slack Alerting](alerting) |
| alerting_threshold | integer | The threshold for triggering alerts [Doc on Slack Alerting](alerting) |
| use_client_credentials_pass_through_routes | boolean | If true, uses client credentials for all pass-through routes. [Doc on pass through routes](pass_through) |
| health_check_details | boolean | If false, hides health check details (e.g. remaining rate limit). [Doc on health checks](health) |
| public_routes | List[str] | (Enterprise Feature) Control list of public routes |
| alert_types | List[str] | Control list of alert types to send to slack (Doc on alert types)[./alerting.md] |
| enforced_params | List[str] | (Enterprise Feature) List of params that must be included in all requests to the proxy |
| enable_oauth2_auth | boolean | (Enterprise Feature) If true, enables oauth2.0 authentication |
| use_x_forwarded_for | str | If true, uses the X-Forwarded-For header to get the client IP address |
| service_account_settings | List[Dict[str, Any]] | Set `service_account_settings` if you want to create settings that only apply to service account keys (Doc on service accounts)[./service_accounts.md] | 
| image_generation_model | str | The default model to use for image generation - ignores model set in request |
| store_model_in_db | boolean | If true, enables storing model + credential information in the DB. |
| store_prompts_in_spend_logs | boolean | If true, allows prompts and responses to be stored in the spend logs table. |
| max_request_size_mb | int | The maximum size for requests in MB. Requests above this size will be rejected. |
| max_response_size_mb | int | The maximum size for responses in MB. LLM Responses above this size will not be sent. |
| proxy_budget_rescheduler_min_time | int | The minimum time (in seconds) to wait before checking db for budget resets. **Default is 597 seconds** |
| proxy_budget_rescheduler_max_time | int | The maximum time (in seconds) to wait before checking db for budget resets. **Default is 605 seconds** |
| proxy_batch_write_at | int | Time (in seconds) to wait before batch writing spend logs to the db. **Default is 10 seconds** |
| proxy_batch_polling_interval | int | Time (in seconds) to wait before polling a batch, to check if it's completed. **Default is 6000 seconds (1 hour)** |
| alerting_args | dict | Args for Slack Alerting [Doc on Slack Alerting](./alerting.md) |
| custom_key_generate | str | Custom function for key generation [Doc on custom key generation](./virtual_keys.md#custom--key-generate) |
| allowed_ips | List[str] | List of IPs allowed to access the proxy. If not set, all IPs are allowed. |
| embedding_model | str | The default model to use for embeddings - ignores model set in request |
| default_team_disabled | boolean | If true, users cannot create 'personal' keys (keys with no team_id). |
| alert_to_webhook_url | Dict[str] | [Specify a webhook url for each alert type.](./alerting.md#set-specific-slack-channels-per-alert-type) |
| key_management_settings | List[Dict[str, Any]] | Settings for key management system (e.g. AWS KMS, Azure Key Vault) [Doc on key management](../secret.md) |
| allow_user_auth | boolean | (Deprecated) old approach for user authentication. |
| user_api_key_cache_ttl | int | The time (in seconds) to cache user api keys in memory. |
| disable_prisma_schema_update | boolean | If true, turns off automatic schema updates to DB |
| litellm_key_header_name | str | If set, allows passing LiteLLM keys as a custom header. [Doc on custom headers](./virtual_keys.md#custom-headers) |
| moderation_model | str | The default model to use for moderation. |
| custom_sso | str | Path to a python file that implements custom SSO logic. [Doc on custom SSO](./custom_sso.md) |
| allow_client_side_credentials | boolean | If true, allows passing client side credentials to the proxy. (Useful when testing finetuning models) [Doc on client side credentials](./virtual_keys.md#client-side-credentials) |
| admin_only_routes | List[str] | (Enterprise Feature) List of routes that are only accessible to admin users. [Doc on admin only routes](./enterprise#control-available-public-private-routes) |
| use_azure_key_vault | boolean | If true, load keys from azure key vault | 
| use_google_kms | boolean | If true, load keys from google kms |
| spend_report_frequency | str | Specify how often you want a Spend Report to be sent (e.g. "1d", "2d", "30d") [More on this](./alerting.md#spend-report-frequency) |
| ui_access_mode | Literal["admin_only"] | If set, restricts access to the UI to admin users only. [Docs](./ui.md#restrict-ui-access) |
| litellm_jwtauth | Dict[str, Any] | Settings for JWT authentication. [Docs](./token_auth.md) |
| litellm_license | str | The license key for the proxy. [Docs](../enterprise.md#how-does-deployment-with-enterprise-license-work) |
| oauth2_config_mappings | Dict[str, str] | Define the OAuth2 config mappings | 
| pass_through_endpoints | List[Dict[str, Any]] | Define the pass through endpoints. [Docs](./pass_through) |
| enable_oauth2_proxy_auth | boolean | (Enterprise Feature) If true, enables oauth2.0 authentication |
| forward_openai_org_id | boolean | If true, forwards the OpenAI Organization ID to the backend LLM call (if it's OpenAI). |
| forward_client_headers_to_llm_api | boolean | If true, forwards the client headers (any `x-` headers and `anthropic-beta` headers) to the backend LLM call |
| maximum_spend_logs_retention_period               | str                   | Used to set the max retention time for spend logs in the db, after which they will be auto-purged                                                                                                                                                                                                                             |
| maximum_spend_logs_retention_interval | str | Used to set the interval in which the spend log cleanup task should run in.                                                                                                                                                                                                                                                   |
### router_settings - Reference

:::info

Most values can also be set via `litellm_settings`. If you see overlapping values, settings on `router_settings` will override those on `litellm_settings`.
:::
```yaml
router_settings:
  routing_strategy: simple-shuffle # Literal["simple-shuffle", "least-busy", "usage-based-routing","latency-based-routing"], default="simple-shuffle" - RECOMMENDED for best performance
  redis_host:            # string
  redis_password:    # string
  redis_port:            # string
  enable_pre_call_checks: true            # bool - Before call is made check if a call is within model context window 
  allowed_fails: 3 # cooldown model if it fails > 1 call in a minute. 
  cooldown_time: 30 # (in seconds) how long to cooldown model if fails/min > allowed_fails
  disable_cooldowns: True                  # bool - Disable cooldowns for all models 
  enable_tag_filtering: True                # bool - Use tag based routing for requests
  retry_policy: {                          # Dict[str, int]: retry policy for different types of exceptions
    "AuthenticationErrorRetries": 3,
    "TimeoutErrorRetries": 3,
    "RateLimitErrorRetries": 3,
    "ContentPolicyViolationErrorRetries": 4,
    "InternalServerErrorRetries": 4
  }
  allowed_fails_policy: {
    "BadRequestErrorAllowedFails": 1000, # Allow 1000 BadRequestErrors before cooling down a deployment
    "AuthenticationErrorAllowedFails": 10, # int 
    "TimeoutErrorAllowedFails": 12, # int 
    "RateLimitErrorAllowedFails": 10000, # int 
    "ContentPolicyViolationErrorAllowedFails": 15, # int 
    "InternalServerErrorAllowedFails": 20, # int 
  }
  content_policy_fallbacks=[{"claude-2": ["my-fallback-model"]}] # List[Dict[str, List[str]]]: Fallback model for content policy violations
  fallbacks=[{"claude-2": ["my-fallback-model"]}] # List[Dict[str, List[str]]]: Fallback model for all errors
```

| Name | Type | Description |
|------|------|-------------|
| routing_strategy | string | The strategy used for routing requests. Options: "simple-shuffle", "least-busy", "usage-based-routing", "latency-based-routing". Default is "simple-shuffle". [More information here](../routing) |
| redis_host | string | The host address for the Redis server. **Only set this if you have multiple instances of LiteLLM Proxy and want current tpm/rpm tracking to be shared across them** |
| redis_password | string | The password for the Redis server. **Only set this if you have multiple instances of LiteLLM Proxy and want current tpm/rpm tracking to be shared across them** |
| redis_port | string | The port number for the Redis server. **Only set this if you have multiple instances of LiteLLM Proxy and want current tpm/rpm tracking to be shared across them**|
| enable_pre_call_check | boolean | If true, checks if a call is within the model's context window before making the call. [More information here](reliability) |
| content_policy_fallbacks | array of objects | Specifies fallback models for content policy violations. [More information here](reliability) |
| fallbacks | array of objects | Specifies fallback models for all types of errors. [More information here](reliability) |
| enable_tag_filtering | boolean | If true, uses tag based routing for requests [Tag Based Routing](tag_routing) |
| cooldown_time | integer | The duration (in seconds) to cooldown a model if it exceeds the allowed failures. |
| disable_cooldowns | boolean | If true, disables cooldowns for all models. [More information here](reliability) |
| retry_policy | object | Specifies the number of retries for different types of exceptions. [More information here](reliability) |
| allowed_fails | integer | The number of failures allowed before cooling down a model. [More information here](reliability) |
| allowed_fails_policy | object | Specifies the number of allowed failures for different error types before cooling down a deployment. [More information here](reliability) |
| default_max_parallel_requests | Optional[int] | The default maximum number of parallel requests for a deployment. |
| default_priority | (Optional[int]) | The default priority for a request. Only for '.scheduler_acompletion()'. Default is None. | 
| polling_interval | (Optional[float]) | frequency of polling queue. Only for '.scheduler_acompletion()'. Default is 3ms. |
| max_fallbacks | Optional[int] | The maximum number of fallbacks to try before exiting the call. Defaults to 5. |
| default_litellm_params | Optional[dict] | The default litellm parameters to add to all requests (e.g. `temperature`, `max_tokens`). |
| timeout | Optional[float] | The default timeout for a request. Default is 10 minutes. |
| stream_timeout | Optional[float] | The default timeout for a streaming request. If not set, the 'timeout' value is used. |
| debug_level | Literal["DEBUG", "INFO"] | The debug level for the logging library in the router. Defaults to "INFO". |
| client_ttl | int | Time-to-live for cached clients in seconds. Defaults to 3600. |
| cache_kwargs | dict | Additional keyword arguments for the cache initialization. |
| routing_strategy_args | dict | Additional keyword arguments for the routing strategy - e.g. lowest latency routing default ttl |
| model_group_alias | dict | Model group alias mapping. E.g. `{"claude-3-haiku": "claude-3-haiku-20240229"}` |
| num_retries | int | Number of retries for a request. Defaults to 3. |
| default_fallbacks | Optional[List[str]] | Fallbacks to try if no model group-specific fallbacks are defined. |
| caching_groups | Optional[List[tuple]] | List of model groups for caching across model groups. Defaults to None. - e.g. caching_groups=[("openai-gpt-3.5-turbo", "azure-gpt-3.5-turbo")]|
| alerting_config | AlertingConfig | [SDK-only arg] Slack alerting configuration. Defaults to None. [Further Docs](../routing.md#alerting-) |
| assistants_config | AssistantsConfig | Set on proxy via `assistant_settings`. [Further docs](../assistants.md) |
| set_verbose | boolean | [DEPRECATED PARAM - see debug docs](./debugging.md) If true, sets the logging level to verbose. |
| retry_after | int | Time to wait before retrying a request in seconds. Defaults to 0. If `x-retry-after` is received from LLM API, this value is overridden. |
| provider_budget_config | ProviderBudgetConfig | Provider budget configuration. Use this to set llm_provider budget limits. example $100/day to OpenAI, $100/day to Azure, etc. Defaults to None. [Further Docs](./provider_budget_routing.md) |
| enable_pre_call_checks | boolean | If true, checks if a call is within the model's context window before making the call. [More information here](reliability) |
| model_group_retry_policy | Dict[str, RetryPolicy] | [SDK-only arg] Set retry policy for model groups. |
| context_window_fallbacks | List[Dict[str, List[str]]] | Fallback models for context window violations. |
| redis_url | str | URL for Redis server. **Known performance issue with Redis URL.** |
| cache_responses | boolean | Flag to enable caching LLM Responses, if cache set under `router_settings`. If true, caches responses. Defaults to False. |
| router_general_settings | RouterGeneralSettings | [SDK-Only] Router general settings - contains optimizations like 'async_only_mode'. [Docs](../routing.md#router-general-settings) |
| optional_pre_call_checks | List[str] | List of pre-call checks to add to the router. Currently supported: 'router_budget_limiting', 'prompt_caching' |
| ignore_invalid_deployments | boolean | If true, ignores invalid deployments. Default for proxy is True - to prevent invalid models from blocking other models from being loaded. |
 
### environment variables - Reference

| Name | Description |
|------|-------------|
| ACTIONS_ID_TOKEN_REQUEST_TOKEN | Token for requesting ID in GitHub Actions
| ACTIONS_ID_TOKEN_REQUEST_URL | URL for requesting ID token in GitHub Actions
| AGENTOPS_ENVIRONMENT | Environment for AgentOps logging integration
| AGENTOPS_API_KEY | API Key for AgentOps logging integration
| AGENTOPS_SERVICE_NAME | Service Name for AgentOps logging integration
| AISPEND_ACCOUNT_ID | Account ID for AI Spend
| AISPEND_API_KEY | API Key for AI Spend
| AIOHTTP_TRUST_ENV | Flag to enable aiohttp trust environment. When this is set to True, aiohttp will respect HTTP(S)_PROXY env vars. **Default is False**
| ALLOWED_EMAIL_DOMAINS | List of email domains allowed for access
| ARIZE_API_KEY | API key for Arize platform integration
| ARIZE_SPACE_KEY | Space key for Arize platform
| ARGILLA_BATCH_SIZE | Batch size for Argilla logging
| ARGILLA_API_KEY | API key for Argilla platform
| ARGILLA_SAMPLING_RATE | Sampling rate for Argilla logging
| ARGILLA_DATASET_NAME | Dataset name for Argilla logging
| ARGILLA_BASE_URL | Base URL for Argilla service
| ATHINA_API_KEY | API key for Athina service
| ATHINA_BASE_URL | Base URL for Athina service (defaults to `https://log.athina.ai`)
| AUTH_STRATEGY | Strategy used for authentication (e.g., OAuth, API key)
| ANTHROPIC_API_KEY | API key for Anthropic service
| ANTHROPIC_API_BASE | Base URL for Anthropic API. Default is https://api.anthropic.com
| AWS_ACCESS_KEY_ID | Access Key ID for AWS services
| AWS_DEFAULT_REGION | Default AWS region for service interactions when AWS_REGION is not set
| AWS_PROFILE_NAME | AWS CLI profile name to be used
| AWS_REGION | AWS region for service interactions (takes precedence over AWS_DEFAULT_REGION)
| AWS_REGION_NAME | Default AWS region for service interactions
| AWS_ROLE_ARN | ARN of the AWS IAM role to assume for authentication
| AWS_ROLE_NAME | Role name for AWS IAM usage
| AWS_SECRET_ACCESS_KEY | Secret Access Key for AWS services
| AWS_SESSION_NAME | Name for AWS session
| AWS_WEB_IDENTITY_TOKEN | Web identity token for AWS
| AWS_WEB_IDENTITY_TOKEN_FILE | Path to file containing web identity token for AWS
| AZURE_API_VERSION | Version of the Azure API being used
| AZURE_AUTHORITY_HOST | Azure authority host URL
| AZURE_CERTIFICATE_PASSWORD | Password for Azure OpenAI certificate
| AZURE_CLIENT_ID | Client ID for Azure services
| AZURE_CLIENT_SECRET | Client secret for Azure services
| AZURE_CODE_INTERPRETER_COST_PER_SESSION | Cost per session for Azure Code Interpreter service
| AZURE_COMPUTER_USE_INPUT_COST_PER_1K_TOKENS | Input cost per 1K tokens for Azure Computer Use service
| AZURE_COMPUTER_USE_OUTPUT_COST_PER_1K_TOKENS | Output cost per 1K tokens for Azure Computer Use service
| AZURE_DEFAULT_RESPONSES_API_VERSION | Version of the Azure Default Responses API being used. Default is "preview"
| AZURE_TENANT_ID | Tenant ID for Azure Active Directory
| AZURE_USERNAME | Username for Azure services, use in conjunction with AZURE_PASSWORD for azure ad token with basic username/password workflow
| AZURE_PASSWORD | Password for Azure services, use in conjunction with AZURE_USERNAME for azure ad token with basic username/password workflow
| AZURE_FEDERATED_TOKEN_FILE | File path to Azure federated token
| AZURE_FILE_SEARCH_COST_PER_GB_PER_DAY | Cost per GB per day for Azure File Search service
| AZURE_SCOPE | For EntraID Auth, Scope for Azure services, defaults to "https://cognitiveservices.azure.com/.default"
| AZURE_KEY_VAULT_URI | URI for Azure Key Vault


BerriAI/litellm
litellm/proxy/_types.py


    )

class ConfigYAML(LiteLLMPydanticObjectBase):
    """
    Documents all the fields supported by the config.yaml
    """

    environment_variables: Optional[dict] = Field(
        None,
        description="Object to pass in additional environment variables via POST request",
    )
    model_list: Optional[List[ModelParams]] = Field(
        None,
        description="List of supported models on the server, with model-specific configs",
    )
    litellm_settings: Optional[dict] = Field(
        None,
        description="litellm Module settings. See __init__.py for all, example litellm.drop_params=True, litellm.set_verbose=True, litellm.api_base, litellm.cache",
    )
    general_settings: Optional[ConfigGeneralSettings] = None
    router_settings: Optional[UpdateRouterConfig] = Field(
        None,
        description="litellm router object settings. See router.py __init__ for all, example router.num_retries=5, router.timeout=5, router.max_retries=5, router.retry_after=5",
    )

    model_config = ConfigDict(protected_namespaces=())

class LiteLLM_VerificationToken(LiteLLMPydanticObjectBase):
    token: Optional[str] = None


BerriAI/litellm
docs/my-website/docs/proxy/configs.md


[**All input params**](https://docs.litellm.ai/docs/completion/input#input-params-1)

**Step 1**: Create a `config.yaml` file
```yaml
model_list:
  - model_name: gpt-4-team1
    litellm_params: # params for litellm.completion() - https://docs.litellm.ai/docs/completion/input#input---request-body
      model: azure/chatgpt-v-2
      api_base: https://openai-gpt-4-test-v-1.openai.azure.com/
      api_version: "2023-05-15"
      azure_ad_token: eyJ0eXAiOiJ
      seed: 12
      max_tokens: 20
  - model_name: gpt-4-team2
    litellm_params:
      model: azure/gpt-4
      api_key: sk-123
      api_base: https://openai-gpt-4-test-v-2.openai.azure.com/
      temperature: 0.2
  - model_name: openai-gpt-4o
    litellm_params:
      model: openai/gpt-4o
      extra_headers: {"AI-Resource Group": "ishaan-resource"}
      api_key: sk-123
      organization: org-ikDc4ex8NB
      temperature: 0.2
  - model_name: mistral-7b
    litellm_params:
      model: ollama/mistral
      api_base: your_ollama_api_base
```

**Step 2**: Start server with config


BerriAI/litellm
docs/my-website/docs/proxy/pass_through.md



### Complete Specification
```yaml
general_settings:
  pass_through_endpoints:
    - path: string                    # Route on LiteLLM Proxy Server
      target: string                  # Target URL for forwarding
      auth: boolean                   # Enable LiteLLM authentication (Enterprise)
      forward_headers: boolean        # Forward all incoming headers
      headers:                        # Custom headers to add
        Authorization: string         # Auth header for target API
        content-type: string         # Request content type
        accept: string               # Expected response format
        LANGFUSE_PUBLIC_KEY: string  # For Langfuse endpoints
        LANGFUSE_SECRET_KEY: string  # For Langfuse endpoints
        : string      # Any custom header
```

### Header Options
- **Authorization**: Authentication for the target API
- **content-type**: Request body format specification  
- **accept**: Expected response format
- **LANGFUSE_PUBLIC_KEY/SECRET_KEY**: For Langfuse integration
- **Custom headers**: Any additional key-value pairs


BerriAI/litellm
litellm/proxy/proxy_server.py


                premium_user = _license_check.is_premium()
        return
    async def load_config(  # noqa: PLR0915
        self, router: Optional[litellm.Router], config_file_path: str
    ):
        """
        Load config values into proxy global state
        """
        global master_key, user_config_file_path, otel_logging, user_custom_auth, user_custom_auth_path, user_custom_key_generate, user_custom_sso, user_custom_ui_sso_sign_in_handler, use_background_health_checks, health_check_interval, use_queue, proxy_budget_rescheduler_max_time, proxy_budget_rescheduler_min_time, ui_access_mode, litellm_master_key_hash, proxy_batch_write_at, disable_spend_logs, prompt_injection_detection_obj, redis_usage_cache, store_model_in_db, premium_user, open_telemetry_logger, health_check_details, callback_settings, proxy_batch_polling_interval

        config: dict = await self.get_config(config_file_path=config_file_path)

        self._load_environment_variables(config=config)

        ## Callback settings
        callback_settings = config.get("callback_settings", {})

        ## LITELLM MODULE SETTINGS (e.g. litellm.drop_params=True,..)
        litellm_settings = config.get("litellm_settings", None)
        if litellm_settings is None:
            litellm_settings = {}
        if litellm_settings:
            # ANSI escape code for blue text
            blue_color_code = "\033[94m"
            reset_color_code = "\033[0m"
            for key, value in litellm_settings.items():
                if key == "cache" and value is True:
                    print(f"{blue_color_code}\nSetting Cache on Proxy")  # noqa
                    from litellm.caching.caching import Cache

                    cache_params = {}
                    if "cache_params" in litellm_settings:
                        cache_params_in_config = litellm_settings["cache_params"]
                        # overwrite cache_params with cache_params_in_config
                        cache_params.update(cache_params_in_config)

                    cache_type = cache_params.get("type", "redis")

                    verbose_proxy_logger.debug("passed cache type=%s", cache_type)

                    if (
                        cache_type == "redis" or cache_type == "redis-semantic"
                    ) and len(cache_params.keys()) == 0:
                        cache_host = get_secret("REDIS_HOST", None)
                        cache_port = get_secret("REDIS_PORT", None)
                        cache_password = None
                        cache_params.update(
                            {
                                "type": cache_type,
                                "host": cache_host,
                                "port": cache_port,
                            }
                        )

                        if get_secret("REDIS_PASSWORD", None) is not None:
                            cache_password = get_secret("REDIS_PASSWORD", None)
                            cache_params.update(
                                {
                                    "password": cache_password,
                                }
                            )

                        # Assuming cache_type, cache_host, cache_port, and cache_password are strings
                        verbose_proxy_logger.debug(
                            "%sCache Type:%s %s",
                            blue_color_code,
                            reset_color_code,
                            cache_type,
                        )
                        verbose_proxy_logger.debug(
                            "%sCache Host:%s %s",
                            blue_color_code,
                            reset_color_code,
                            cache_host,
                        )
                        verbose_proxy_logger.debug(
                            "%sCache Port:%s %s",
                            blue_color_code,
                            reset_color_code,
                            cache_port,
                        )
                        verbose_proxy_logger.debug(
                            "%sCache Password:%s %s",
                            blue_color_code,
                            reset_color_code,
                            cache_password,
                        )

