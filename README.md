# CockroachDB Natural Language to SQL Converter

A specialized skill for converting natural language queries and requirements into optimized CockroachDB SQL, with the ability to execute queries directly against CockroachDB instances.

## Overview

This skill transforms natural language descriptions into CockroachDB-compliant SQL queries and can optionally execute them against a connected database. It ensures all generated code follows CockroachDB's distributed architecture best practices, automatically applies CockroachDB-specific optimizations, and avoids common pitfalls.

## Installation

1. Clone or download this repository in a directory 
```
git clone https://github.com/dshjoshi/cockroachdb-sql-skills.git
```
2. Create ~/.claude/skills/cockroachdb-sql directory
```
mkdir ~/.claude/skills/cockroachdb-sql
```
3. Copy the content of the directory where report is cloned or downloaded to  ./claude/skills/cockroachdb-sql
```
cp <skill-download-location>/cockroachdb-sql/* ~/.claude/skills/cockroachdb-sql/
```

## Database Connection & Execution

The skill supports three methods to connect and execute queries (in priority order):

### 1. User-Provided URL in Prompt
Include a connection URL directly in your message:
```
"Connect to postgresql://root@localhost:26257/mydb and create a users table"
```

### 2. Cockroach-Cloud MCP Server
If you have the `cockroach-cloud` MCP server connected, the skill will automatically use it for execution.

### 3. Environment Variable
Set the `COCKROACH_URL` environment variable:
```bash
export COCKROACH_URL="postgresql://user:password@host:26257/database"
```

If none of these methods are available, the skill will generate SQL with instructions on how to connect.

## Features

- **Natural Language Processing**: Converts plain English descriptions into SQL queries
- **CockroachDB Optimization**: Automatically applies distributed database best practices
- **Type Normalization**: Converts standard SQL types to CockroachDB-optimized equivalents
- **Performance Tuning**: Includes appropriate indexes, partitioning, and query patterns
- **Multi-Region Support**: Handles global distribution patterns when needed
- **Time Travel Queries**: Supports AS OF SYSTEM TIME for historical data access

## Quick Start

### Generate SQL Only (No Execution)
```
/cockroachdb-sql Create a users table with email and password
```

### Execute with User-Provided URL
```
Connect to postgresql://root@localhost:26257/mydb and execute:
Create a users table with email, password, and profile data
```

### Execute with Environment Variable
```bash
# First, set the environment variable
export COCKROACH_URL="postgresql://root@localhost:26257/defaultdb"

# Then use the skill
/crdb-sql Execute: Create an orders table with customer reference
```

### Execute with MCP Server
```
# With cockroach-cloud MCP connected
/cockroachdb-sql Run query: Show me all active user sessions
```

## Usage

### Trigger Commands
- `/cockroachdb-sql` - Convert natural language to SQL
- `/crdb-sql` - Generate CockroachDB-specific SQL
- Or simply ask: "CockroachDB SQL for" or "Generate CockroachDB query"

### Execution Keywords
Use these keywords to indicate you want to execute the SQL:
- **Execute**: "Execute: create a products table"
- **Run**: "Run query to find recent orders"
- **Apply**: "Apply this schema to my database"
- **Create in database**: "Create a users table in database"

### Example Prompts
```
# Generation only
"Create a users table with email and password"
"Generate SQL for finding orders from last week"

# With execution
"Execute: Create a products table with inventory tracking"
"Run query to show database statistics"
"Connect to postgresql://localhost:26257/shop and create schema"
```

## Key Transformations

The skill automatically applies these critical transformations:

| Standard SQL | CockroachDB Optimization |
|-------------|-------------------------|
| `TEXT` | `STRING` |
| `TIMESTAMP` | `TIMESTAMPTZ` |
| `JSON` | `JSONB` |
| `INT`/`INTEGER` | `INT8` (64-bit) |
| `AUTO_INCREMENT` | `UUID DEFAULT gen_random_uuid()` |
| No PRIMARY KEY | Always adds PRIMARY KEY |

## Directory Structure

```
crdb-v1-sql-skills/
├── README.md                       # This file
├── SKILL.md                        # Core skill definition (~80 lines)
├── CLAUDE.md                       # Development workflow and preferences
├── .env.example                    # Environment configuration template
├── test-connection.sh              # Connection testing script
├── cockroachdb-rules/              # Consolidated SQL conversion rules (6 files)
│   ├── README.md                   # Navigation and quick reference
│   ├── 00-fundamental-principles.md # Core rules that apply to ALL
│   ├── 01-schema-design.md         # Tables, indexes, constraints
│   ├── 02-dml-operations.md        # INSERT, UPDATE, DELETE, transactions
│   ├── 03-query-patterns.md        # SELECT, JOINs, CTEs, JSON/arrays
│   ├── 04-optimization.md          # Performance and anti-patterns
│   └── 05-operational.md           # Admin, backup, monitoring
├── docs/                           # Supporting documentation
│   ├── CONNECTION.md               # Database connection methods
│   ├── EXAMPLES.md                 # SQL examples and patterns
│   └── EXECUTION.md                # Implementation details
└── .claude/                        # Claude settings
    └── settings.local.json
```

### Rules Organization

The skill's rules are consolidated into 6 focused files:

1. **00-fundamental-principles.md** - Core rules that always apply
2. **01-schema-design.md** - Table creation, indexes, constraints
3. **02-dml-operations.md** - Data manipulation and transactions
4. **03-query-patterns.md** - Query construction and patterns
5. **04-optimization.md** - Performance tuning and anti-patterns
6. **05-operational.md** - Admin and maintenance operations

See `cockroachdb-rules/README.md` for detailed navigation guide.

## Output Format

The skill generates responses with:
1. **SQL Code**: Properly formatted CockroachDB SQL with explanatory comments
2. **Explanation**: Design choices and CockroachDB-specific features used
3. **Performance Considerations**: Index recommendations and optimization suggestions
4. **Alternative Approaches**: When multiple valid solutions exist

### Example Output
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email STRING NOT NULL UNIQUE,
  password_hash STRING NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),

  -- Index for login queries
  INDEX idx_email (email)
);
```

## Advanced Features

### Multi-Region Support
Handles LOCALITY hints for global applications:
```sql
CREATE TABLE global_settings (
  key STRING PRIMARY KEY,
  value JSONB
) LOCALITY GLOBAL;
```

### Time Travel Queries
Supports historical data access:
```sql
SELECT * FROM orders AS OF SYSTEM TIME '-1h';
```

### Vector Search
AI/ML embeddings with similarity search:
```sql
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  embedding VECTOR(1536),
  INDEX idx_embedding USING HNSW(embedding vector_cosine_ops)
);
```

## Best Practices Applied

The skill automatically ensures:
- ✅ Every table has a PRIMARY KEY
- ✅ UUIDs for distributed write performance
- ✅ Appropriate indexes for query patterns
- ✅ UPSERT for blind writes
- ✅ STORING columns for covering indexes
- ✅ Follower reads for read-heavy workloads
- ✅ RETURNING NOTHING for write-only operations
- ✅ Proper transaction isolation levels

## Anti-Patterns Avoided

The skill prevents common mistakes:
- ❌ Missing PRIMARY KEY
- ❌ Using AUTO_INCREMENT
- ❌ TEXT instead of STRING
- ❌ TIMESTAMP without timezone
- ❌ JSON instead of JSONB
- ❌ SELECT * in production queries
- ❌ Inefficient existence checks with COUNT(*)
- ❌ Missing indexes on foreign keys

## Requirements

- CockroachDB v24.x or later
- Understanding of basic SQL concepts
- (Optional) Knowledge of distributed database concepts for advanced features

## Common Workflows

### Schema Development
```
User: "I need a schema for an e-commerce platform"

Skill:
1. Generates complete schema
2. Asks if you want to execute
3. If yes, creates all tables with proper relationships
4. Shows confirmation for each table
5. Provides sample queries
```

### Query Optimization
```
User: "Execute and explain: Find top 10 customers by order value"

Skill:
1. Generates optimized query
2. Runs EXPLAIN to show query plan
3. Executes actual query
4. Shows results and performance metrics
5. Suggests indexes if needed
```

## Testing Your Connection

```bash
# Test environment variable connection
./test-connection.sh

# Test with specific URL
./test-connection.sh "postgresql://user:pass@host:26257/db"

# Validate generated SQL syntax
cockroach sql --url="$COCKROACH_URL" -e "EXPLAIN [GENERATED_SQL]"
```

## Troubleshooting

### Connection Issues
- **"Connection refused"**: Check if CockroachDB is running
- **"Authentication failed"**: Verify credentials in connection string
- **"Database does not exist"**: Create database first or use defaultdb
- **"SSL required"**: Add `?sslmode=require` to connection string

### Safety Features
The skill will warn before:
- `DROP TABLE` or `DROP DATABASE`
- `TRUNCATE TABLE`
- `DELETE` without WHERE clause
- Schema changes that might lose data

## Version

**Current Version**: 1.0.0
- Full CockroachDB v24.x+ support
- Type normalization and optimization
- Multi-region and time travel support
- Vector search capabilities

## Author

CockroachDB Team

## License

This skill is part of the CockroachDB ecosystem and follows CockroachDB's standard licensing.

## Resources

- [CockroachDB Documentation](https://www.cockroachlabs.com/docs/stable/)
- [SQL Style Guide](https://www.cockroachlabs.com/docs/stable/sql-style-guide.html)
- [Performance Best Practices](https://www.cockroachlabs.com/docs/stable/performance-best-practices-overview.html)

## Contributing

To improve this skill:
1. Update rule files in `cockroachdb-rules/` for new patterns
2. Modify `SKILL.md` for behavioral changes
3. Test with various natural language inputs
4. Ensure generated SQL follows CockroachDB best practices

## Support

For issues or questions:
- Review the rules in `cockroachdb-rules/` directory
- Check `SKILL.md` for detailed documentation
- Consult CockroachDB official documentation
- Open an issue in the repository
