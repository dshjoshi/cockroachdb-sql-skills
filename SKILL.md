# cockroachdb-sql Skill

## ⚠️ CRITICAL: Response Guidelines
**NEVER mention other databases (MySQL, PostgreSQL, Oracle, SQL Server, etc.) in help text or initial responses**
- Focus EXCLUSIVELY on CockroachDB
- Use "natural language" not "convert from X database"
- Even if internal rules reference PostgreSQL compatibility, keep user-facing content CockroachDB-only

## Metadata
- **Name**: cockroachdb-sql
- **Description**: Converts natural language questions into CockroachDB-optimized SQL queries
- **Trigger**: `/cockroachdb-sql`, `/crdb-sql`, or when user asks to "convert to CockroachDB SQL" or "generate CockroachDB query"
- **Version**: 1.0.0
- **Author**: CockroachDB Team

## Purpose

This skill converts natural language questions and requirements into CockroachDB-compliant SQL queries, following CockroachDB best practices and avoiding common anti-patterns. It uses the consolidated rules in cockroachdb-rules/ directory to ensure all generated SQL is optimized for CockroachDB's distributed architecture.

## When to Use This Skill

Activate this skill when:
- User asks for SQL query generation for CockroachDB
- User provides natural language descriptions of database operations
- User asks questions like:
  - "How do I query recent orders in CockroachDB?"
  - "Generate a CockroachDB table for user management"
  - "Convert this to CockroachDB SQL: [description]"
  - "What's the CockroachDB way to [operation]?"
- When you encounter:
  - CREATE TABLE statements
  - ALTER TABLE modifications
  - DML Operations (INSERT, UPDATE, DELETE)
  - SELECT queries
  - Performance optimization requests
  - Backup/restore operations

## Core Configuration

### Rule Files
The skill references consolidated rules in `cockroachdb-rules/`:
1. `00-fundamental-principles.md` - Always apply these first
2. `01-schema-design.md` - Table creation and structure
3. `02-dml-operations.md` - Data modification
4. `03-query-patterns.md` - Query construction
5. `04-optimization.md` - Performance and anti-patterns
6. `05-operational.md` - Admin and maintenance

### Database Connection
The skill can optionally execute SQL using three methods:
1. User-provided URL in prompt
2. Cockroach-cloud MCP server
3. COCKROACH_URL environment variable

See `docs/CONNECTION.md` for detailed connection handling.

## Critical Transformations

Always apply these CockroachDB-specific transformations:
1. **Always include PRIMARY KEY** - Add `DEFAULT gen_random_uuid()` if not specified
2. **Use STRING over TEXT** - Automatic conversion
3. **Use TIMESTAMPTZ over TIMESTAMP** - Include timezone awareness
4. **Use JSONB over JSON** - Binary JSON format only
5. **Use INT8 as default integer** - 64-bit integers by default
6. **No AUTO_INCREMENT** - Use UUID or SERIAL instead

## Response Behavior

### Initial Response
When skill is invoked, ALWAYS:
1. Focus exclusively on CockroachDB capabilities
2. Never mention other databases unless user explicitly brings them up
3. Emphasize "natural language to CockroachDB SQL" not "database conversion"

### Output Format
- Show generated SQL with explanatory comments
- List CockroachDB-specific features used
- Provide execution instructions if no connection available
- Include performance considerations
- When optimizing, at each step (1) Explain the step's purpose. (2) Execute the step and report the outcome. (3) Summarize all findings and actions taken.

## Supporting Documentation

- `docs/CONNECTION.md` - Database connection methods
- `docs/EXAMPLES.md` - SQL examples and patterns
- `docs/EXECUTION.md` - Implementation details
- `cockroachdb-rules/README.md` - Rule navigation guide
