# Implementation & Execution Logic

## Implementation Logic for Database Execution

### 1. Detecting User-Provided Connection URL

```javascript
// Pattern to extract CockroachDB/PostgreSQL URLs from user prompt
const urlPattern = /(?:postgresql|cockroachdb):\/\/[^\s]+/i;
const match = userPrompt.match(urlPattern);
if (match) {
  connectionUrl = match[0];
  connectionMethod = "user-provided";
}
```

### 2. Checking for MCP Server Availability

When skill is invoked:
1. Check if `cockroach-cloud` tool is available in MCP tools list
2. If available, use MCP methods to execute queries:
   ```
   Use cockroach-cloud.query tool with parameters:
   - sql: [generated SQL]
   - cluster: [optional cluster name]
   ```

### 3. Checking Environment Variable

```bash
# Check for COCKROACH_URL environment variable
if [ -n "$COCKROACH_URL" ]; then
  echo "Found COCKROACH_URL environment variable"
  # Use this for connection
fi
```

### 4. Execution Decision Tree

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

## Input Processing Pipeline

### 1. Parse Natural Language Intent
- Identify the operation type (SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, etc.)
- Extract entities (tables, columns, conditions)
- Determine relationships and joins
- Identify performance requirements
- **Check for execution intent** (execute/run/apply vs generate/show)
- **Extract connection URL if provided** (look for postgresql:// or cockroachdb:// patterns)

### 2. Connection Detection
- Parse prompt for connection strings (postgresql://... or cockroachdb://...)
- Check if cockroach-cloud MCP server is available using tool discovery
- Check for COCKROACH_URL environment variable using Bash: `echo $COCKROACH_URL`
- Store active connection method for session

### 3. Context Gathering
- Check for existing schema context in conversation
- If connected to database, optionally query existing schema:
  - `SHOW TABLES;` to see existing tables
  - `SHOW CREATE TABLE table_name;` for existing structure
- Ask clarifying questions if needed:
  - Table structure if not provided
  - Data types for columns
  - Index requirements
  - Multi-region needs
  - Performance characteristics

### 4. Apply CockroachDB Rules
- Reference cockroachdb-rules/ directory for all conversions
- Ensure compliance with CockroachDB best practices
- **Determine rule category based on operation**:
  * CREATE/ALTER TABLE → Schema Design
  * INSERT/UPDATE/DELETE/UPSERT → DML Operations
  * SELECT/JOIN/Aggregation → Read Queries
  * "slow query"/"optimize" → Query Optimization
  * BACKUP/RESTORE/CHANGEFEED → Operational
- **Always apply fundamental principles** regardless of category

## Output Format Templates

### When Generating SQL Only (No Connection)

```markdown
## Generated CockroachDB SQL

```sql
-- Generated query with comments explaining key decisions
[SQL QUERY HERE]
```

## Explanation
- **Design Choices**: [Why specific CockroachDB features were used]
- **Performance Considerations**: [Index recommendations, partitioning suggestions]
- **Alternative Approaches**: [If applicable]

## CockroachDB-Specific Features Used
- [List features like AS OF SYSTEM TIME, UPSERT, etc.]

## To Execute This Query
You can run this SQL by:
1. Setting COCKROACH_URL environment variable: `export COCKROACH_URL="your-connection-string"`
2. Connecting via cockroach-cloud MCP server
3. Or provide connection URL in your next message
```

### When Executing Against Database

```markdown
## Connection Method
Using: [cockroach-cloud MCP / Environment Variable / User-provided URL]

## Executed SQL

```sql
[SQL QUERY HERE]
```

## Execution Results
✅ Query executed successfully
- Rows affected: [number]
- Execution time: [time]

[Show actual results for SELECT queries]

## Next Steps
- [Suggest related operations based on what was just executed]
```

### Multiple Query Response

When generating multiple related queries, organize them logically:

```markdown
## 1. Schema Creation

```sql
-- Table creation with CockroachDB best practices
```

## 2. Index Creation

```sql
-- Performance-optimized indexes
```

## 3. Sample Operations

```sql
-- INSERT, SELECT, UPDATE examples
```
```

## Validation Steps

1. **Schema Validation**
   - Verify all tables have PRIMARY KEY
   - Check data type compatibility
   - Validate constraint definitions

2. **Query Validation**
   - Ensure proper JOIN conditions
   - Verify WHERE clause syntax
   - Check for ambiguous column references

3. **Performance Validation**
   - Look for missing indexes
   - Check for full table scans
   - Validate transaction isolation needs

4. **CockroachDB-Specific Validation**
   - Ensure distributed performance patterns
   - Check for appropriate use of UUID keys
   - Validate multi-region configurations

## Error Handling

### Connection Errors
```javascript
try {
  // Attempt connection
} catch (error) {
  if (error.code === 'ECONNREFUSED') {
    return "Unable to connect. Please ensure CockroachDB is running.";
  } else if (error.code === '28P01') {
    return "Authentication failed. Please check credentials.";
  } else {
    return `Connection error: ${error.message}`;
  }
}
```

### SQL Execution Errors
```javascript
try {
  // Execute SQL
} catch (error) {
  if (error.code === '42P01') {
    return "Table does not exist. Please create it first.";
  } else if (error.code === '23505') {
    return "Duplicate key violation. Record already exists.";
  } else {
    return `SQL error: ${error.message}`;
  }
}
```

## Performance Monitoring

### Query Performance Tracking
```sql
-- Before execution
EXPLAIN ANALYZE [query];

-- After execution
SHOW LAST QUERY STATISTICS;
```

### Connection Pool Management
```javascript
const poolConfig = {
  max: 20,           // Maximum connections
  min: 5,            // Minimum connections
  idleTimeout: 30000 // Idle timeout in ms
};
```

## Security Implementation

### SQL Injection Prevention
```javascript
// Always use parameterized queries
const query = 'SELECT * FROM users WHERE email = $1';
const values = [userEmail];
// Execute with parameters, never string concatenation
```

### Connection String Sanitization
```javascript
function sanitizeConnectionUrl(url) {
  // Remove password from logging
  return url.replace(/:([^@]+)@/, ':***@');
}
```

## Testing Implementation

### Unit Tests for SQL Generation
```javascript
test('should convert TEXT to STRING', () => {
  const input = 'CREATE TABLE test (description TEXT)';
  const output = convertToCockroachDB(input);
  expect(output).toContain('description STRING');
});
```

### Integration Tests for Execution
```javascript
test('should execute CREATE TABLE', async () => {
  const sql = 'CREATE TABLE test_table (id UUID PRIMARY KEY)';
  const result = await executeSQL(sql);
  expect(result.success).toBe(true);
});
```

## Debugging Features

### Verbose Mode
When debugging is needed:
```sql
-- Show query plan
EXPLAIN (VERBOSE, TYPES) [query];

-- Show distribution
EXPLAIN (DISTSQL) [query];

-- Full analysis
EXPLAIN (ANALYZE, VERBOSE, DISTSQL) [query];
```

### Logging
```javascript
function logExecution(sql, result, timing) {
  console.log({
    timestamp: new Date().toISOString(),
    sql: sql.substring(0, 100), // First 100 chars
    success: result.success,
    rowCount: result.rowCount,
    executionTime: timing
  });
}
```