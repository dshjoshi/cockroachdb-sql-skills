# CockroachDB Rules Guide

## Quick Navigation

This directory contains consolidated CockroachDB SQL rules organized into 6 focused files:

1. **[00-fundamental-principles.md](00-fundamental-principles.md)** - Core rules that apply to ALL operations
2. **[01-schema-design.md](01-schema-design.md)** - Tables, indexes, constraints, partitioning, RLS
3. **[02-dml-operations.md](02-dml-operations.md)** - INSERT, UPDATE, DELETE, UPSERT, transactions
4. **[03-query-patterns.md](03-query-patterns.md)** - SELECT, JOINs, CTEs, window functions, JSON/arrays
5. **[04-optimization.md](04-optimization.md)** - Performance tuning and anti-patterns
6. **[05-operational.md](05-operational.md)** - Admin, backup/restore, changefeeds, monitoring

## Decision Flow

```
1. ALWAYS start with 00-fundamental-principles.md
   ↓
2. Identify task type from request
   ↓
3. Apply relevant category file(s)
   ↓
4. Check 04-optimization.md if performance mentioned
   ↓
5. Validate against anti-patterns in 04-optimization.md
```

## Task-to-File Mapping

### Creating Tables/Schema
**Keywords**: create table, alter table, data type, primary key, foreign key, constraint
- → **01-schema-design.md** (complete schema patterns)
- → **00-fundamental-principles.md** (type normalization)

### Data Modification
**Keywords**: insert, update, delete, upsert, bulk, transaction
- → **02-dml-operations.md** (all DML patterns)
- → **04-optimization.md** (bulk operation tips)

### Writing Queries
**Keywords**: select, find, get, join, aggregate, group by, window, CTE
- → **03-query-patterns.md** (comprehensive query patterns)
- → **00-fundamental-principles.md** (type conversions)

### JSON/Array Operations
**Keywords**: json, jsonb, array, unnest, contains
- → **03-query-patterns.md** (JSON and array sections)

### Performance Issues
**Keywords**: slow, optimize, performance, explain, index
- → **04-optimization.md** (performance patterns and anti-patterns)
- → **01-schema-design.md** (index patterns section)

### Database Administration
**Keywords**: backup, restore, monitor, admin, changefeed, show
- → **05-operational.md** (all operational tasks)

## Common Request Patterns

| User Request | Primary File | Secondary Files |
|-------------|--------------|-----------------|
| "Create a users table" | 01-schema-design | 00-fundamental-principles |
| "Insert 10000 records efficiently" | 02-dml-operations | 04-optimization |
| "Query with time travel" | 03-query-patterns | - |
| "Extract JSON field from column" | 03-query-patterns | - |
| "Query is running slow" | 04-optimization | 01-schema-design (indexes) |
| "Set up backup strategy" | 05-operational | - |
| "Add index on email column" | 01-schema-design | 04-optimization |
| "Recursive query for hierarchy" | 03-query-patterns | - |
| "Create changefeed to Kafka" | 05-operational | - |
| "Design high-performance schema" | 01-schema-design | 00-fundamental, 04-optimization |

## Multi-Category Scenarios

### Scenario: "Design and populate user table"
1. Start with **00-fundamental-principles.md** (type rules)
2. Design with **01-schema-design.md** (table creation)
3. Populate with **02-dml-operations.md** (bulk insert)

### Scenario: "Migrate data from old table"
1. Query with **03-query-patterns.md** (SELECT patterns)
2. Insert with **02-dml-operations.md** (INSERT...SELECT)
3. Optimize with **04-optimization.md** (batch operations)

### Scenario: "Set up production database"
1. Design with **01-schema-design.md** (schema patterns)
2. Configure with **05-operational.md** (zone configs)
3. Backup with **05-operational.md** (backup strategy)

## Quick Reference Checklist

### ✅ Always Apply
- [ ] **00-fundamental-principles.md** - Core type conversions
- [ ] Check against anti-patterns in **04-optimization.md**

### ✅ For Schema Work
- [ ] **01-schema-design.md** - Complete schema guide
- [ ] UUID primary keys (not sequential)
- [ ] STORING clause on indexes
- [ ] Proper data types (STRING not TEXT, TIMESTAMPTZ not TIMESTAMP)

### ✅ For Data Operations
- [ ] **02-dml-operations.md** - DML patterns
- [ ] UPSERT for blind writes
- [ ] RETURNING NOTHING when results not needed
- [ ] Batch operations for large datasets

### ✅ For Queries
- [ ] **03-query-patterns.md** - Query patterns
- [ ] AS OF SYSTEM TIME for historical data
- [ ] CTEs for complex logic
- [ ] Proper JSON/array operators

### ✅ For Performance
- [ ] **04-optimization.md** - Performance guide
- [ ] EXPLAIN ANALYZE for slow queries
- [ ] Avoid full table scans
- [ ] Use prepared statements

### ✅ For Operations
- [ ] **05-operational.md** - Admin guide
- [ ] Regular backups configured
- [ ] Monitoring queries in place
- [ ] Changefeeds for real-time data

## File Structure

```
cockroachdb-rules/
├── README.md                        # This file - Navigation guide
├── 00-fundamental-principles.md     # Core rules (50 lines)
├── 01-schema-design.md              # Schema patterns (300 lines)
├── 02-dml-operations.md             # DML operations (150 lines)
├── 03-query-patterns.md             # Query patterns (350 lines)
├── 04-optimization.md               # Performance (150 lines)
└── 05-operational.md                # Admin tasks (150 lines)
```

## Usage in SQL Generation

When generating CockroachDB SQL:

1. **Type Conversion**: Always apply rules from 00-fundamental-principles.md
2. **Pattern Selection**: Choose patterns from relevant category file
3. **Optimization Check**: Validate with 04-optimization.md
4. **Anti-pattern Scan**: Ensure no anti-patterns from 04-optimization.md

## Examples

### Example: User asks "Create a high-performance users table"

```sql
-- Apply from 00-fundamental-principles.md: UUID, TIMESTAMPTZ
-- Apply from 01-schema-design.md: Table structure, indexes
-- Apply from 04-optimization.md: STORING clause, avoid anti-patterns

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  -- UUID for distribution
    email STRING NOT NULL UNIQUE,                   -- STRING not TEXT
    created_at TIMESTAMPTZ DEFAULT now(),           -- TIMESTAMPTZ not TIMESTAMP
    metadata JSONB,                                  -- JSONB not JSON
    INDEX idx_created (created_at DESC) STORING (email)  -- Covering index
);
```

### Example: User asks "Bulk insert user data"

```sql
-- Apply from 02-dml-operations.md: Bulk insert pattern
-- Apply from 04-optimization.md: RETURNING NOTHING

INSERT INTO users (email, name) VALUES
    ('user1@example.com', 'User 1'),
    ('user2@example.com', 'User 2'),
    -- ... up to 1000 rows
    ('user1000@example.com', 'User 1000')
RETURNING NOTHING;  -- Performance optimization
```

## Version

These rules apply to CockroachDB v24.x and later versions.