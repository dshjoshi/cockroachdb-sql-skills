# Database Connection & Execution

This skill can **optionally execute** generated SQL queries against a CockroachDB instance using one of three methods:

## Connection Methods (in priority order)

### 1. User-Provided URL in Prompt
- If user includes a connection URL in their message (e.g., `postgresql://user:pass@host:26257/db`)
- Extract and use this URL for the current session
- Example: "Connect to postgresql://root@localhost:26257/defaultdb and create a users table"

### 2. Cockroach-Cloud MCP Server
- Check if `cockroach-cloud` MCP server is available
- Use MCP tools to execute queries if connected
- This provides secure, authenticated access to CockroachDB clusters

### 3. Environment Variable (COCKROACH_URL)
- Check for `COCKROACH_URL` environment variable
- Use this as default connection if no other method is specified
- Example: `export COCKROACH_URL="postgresql://root@localhost:26257/defaultdb"`

## Execution Flow

When user requests database operations:

### 1. Detect Intent
- Keywords like "execute", "run", "apply", "create in database" indicate execution desire
- Keywords like "generate", "show me SQL", "what would the query be" indicate generation only

### 2. Check Connection Availability
```
IF user provides URL in prompt:
    Use provided URL
ELSE IF cockroach-cloud MCP is available:
    Use MCP connection
ELSE IF COCKROACH_URL env var exists:
    Use environment URL
ELSE:
    Generate SQL only with connection instructions
```

### 3. Execute or Generate
- If connection available AND user wants execution: Execute and show results
- Otherwise: Show generated SQL with appropriate connection instructions

## Connection Method Behaviors

### User-Provided URL
When user includes URL in their prompt:
1. Extract connection URL from prompt using pattern: `/(?:postgresql|cockroachdb):\/\/[^\s]+/i`
2. Store for current session
3. Connect to database
4. Execute SQL statements
5. Show results and confirmation

### MCP Server Connection
When cockroach-cloud MCP is available:
1. Check for tool availability in MCP tools list
2. Use MCP methods to execute queries:
   ```
   Use cockroach-cloud.query tool with parameters:
   - sql: [generated SQL]
   - cluster: [optional cluster name]
   ```
3. Handle responses and display results

### Environment Variable
When using COCKROACH_URL:
1. Check for variable using: `echo $COCKROACH_URL`
2. Validate connection string format
3. Use for database operations
4. Cache for session duration

## Execution Decision Tree
```
IF prompt contains "execute", "run", "apply to database":
  IF connection_url_in_prompt:
    -> Use provided URL to execute
  ELIF cockroach_cloud_mcp_available:
    -> Execute via MCP server
  ELIF COCKROACH_URL_env_exists:
    -> Execute using environment URL
  ELSE:
    -> Generate SQL with instructions on how to connect
ELSE:
  -> Generate SQL only (default behavior)
```

## Handling Connection Errors

If connection fails:
1. Show the generated SQL anyway
2. Provide troubleshooting steps:
   - Verify connection string format
   - Check network connectivity
   - Ensure CockroachDB cluster is running
   - Verify credentials
3. Offer alternative connection methods

## Security Considerations

### Connection String Safety
- Never store credentials in plain text
- Use environment variables for sensitive data
- Prefer MCP server for production connections

### Query Execution Safety
- Always validate SQL before execution
- Use prepared statements when possible
- Limit permissions for connection user
- Never execute destructive operations without confirmation

## Testing Connection

To test if a connection works:
```bash
#!/bin/bash
# test-connection.sh
if [ -z "$COCKROACH_URL" ]; then
  echo "COCKROACH_URL environment variable not set"
  exit 1
fi

cockroach sql --url="$COCKROACH_URL" --execute="SELECT version();"
```

## Connection String Format

CockroachDB uses PostgreSQL-compatible connection strings:

```
postgresql://[user[:password]@][host][:port][/database][?parameters]
```

Examples:
```
# Local insecure cluster
postgresql://root@localhost:26257/defaultdb?sslmode=disable

# Secure cloud cluster
postgresql://user:password@cluster-name-123.aws-region.cockroachlabs.cloud:26257/defaultdb?sslmode=require

# With connection pool
postgresql://user@host:26257/db?pool_max_conns=10
```

## Common Parameters

- `sslmode`: Security level (disable, require, verify-ca, verify-full)
- `application_name`: Identify your application
- `pool_max_conns`: Maximum connections in pool
- `connect_timeout`: Connection timeout in seconds