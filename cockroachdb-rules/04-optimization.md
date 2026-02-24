# OPTIMIZATION

## Performance Patterns

### Query Hints
```sql
-- Force index usage
SELECT * FROM orders@orders_user_id_idx WHERE user_id = 1;

-- Force primary key scan
SELECT * FROM orders@primary WHERE id > 1000;

-- Force specific join type
SELECT * FROM orders
INNER HASH JOIN users ON orders.user_id = users.id;

SELECT * FROM orders
INNER MERGE JOIN users ON orders.user_id = users.id;

-- Disable automatic statistics
SET CLUSTER SETTING sql.stats.automatic_collection.enabled = false;

-- Force statistics refresh
CREATE STATISTICS stats_name ON column FROM table;
```

### Range Operations
```sql
-- Split ranges at specific values (admin operation)
ALTER TABLE orders SPLIT AT VALUES (1000), (2000), (3000);

-- Split with expiration
ALTER TABLE events SPLIT AT VALUES ('2024-01-01')
  WITH EXPIRATION '2024-02-01';

-- Unsplit ranges
ALTER TABLE orders UNSPLIT AT VALUES (1000);
ALTER TABLE orders UNSPLIT ALL;

-- Scatter data (redistribute)
ALTER TABLE large_table SCATTER;

-- Scatter with specific range
ALTER TABLE orders SCATTER FROM (1) TO (1000);
```

### Zone Configuration
```sql
-- Configure replication
ALTER TABLE important_data CONFIGURE ZONE USING
  num_replicas = 5,
  constraints = '{"+region=us-east": 2, "+region=us-west": 2}',
  lease_preferences = '[[+region=us-east]]';

-- Configure garbage collection
ALTER TABLE logs CONFIGURE ZONE USING
  gc.ttlseconds = 3600;  -- 1 hour

-- Configure range size
ALTER TABLE large_table CONFIGURE ZONE USING
  range_max_bytes = 134217728,  -- 128MB
  range_min_bytes = 16777216;   -- 16MB
```

### Query Optimization Techniques

#### Use RETURNING NOTHING
```sql
-- When you don't need results
INSERT INTO logs (data) VALUES ('entry') RETURNING NOTHING;
UPDATE large_table SET processed = true RETURNING NOTHING;
DELETE FROM old_data WHERE date < '2023-01-01' RETURNING NOTHING;
```

#### Batch Operations
```sql
-- Batch inserts
INSERT INTO items (id, name) VALUES
  (gen_random_uuid(), 'Item 1'),
  (gen_random_uuid(), 'Item 2'),
  -- ... up to 1000 rows
  (gen_random_uuid(), 'Item 1000');

-- Batch updates with LIMIT
UPDATE events
SET processed = true
WHERE processed = false
ORDER BY created_at
LIMIT 1000;
```

#### Covering Indexes
```sql
-- Add STORING clause for covering queries
CREATE INDEX idx_orders_user ON orders (user_id)
STORING (total, status, created_at);

-- Query uses only index (no table lookup)
SELECT user_id, total, status FROM orders WHERE user_id = $1;
```

#### Partial Indexes
```sql
-- Index only active records
CREATE INDEX idx_active_users ON users (email)
WHERE deleted_at IS NULL;

-- Index recent data
CREATE INDEX idx_recent_events ON events (created_at)
WHERE created_at > '2024-01-01';
```

## Common Anti-Patterns

### DON'T Use These
```sql
-- AUTO_INCREMENT (not supported)
id INT AUTO_INCREMENT  -- ❌ Will fail

-- CREATE TABLE without PRIMARY KEY
CREATE TABLE bad (data STRING);  -- ❌ Creates hidden rowid

-- TEXT type (use STRING)
description TEXT  -- ❌ Use STRING instead

-- JSON type (use JSONB)
data JSON  -- ❌ Always use JSONB

-- TIMESTAMP without timezone
created_at TIMESTAMP  -- ❌ Use TIMESTAMPTZ

-- TRUNCATE with CASCADE on production
TRUNCATE TABLE users CASCADE;  -- ❌ Dangerous

-- Unqualified DELETE
DELETE FROM table;  -- ❌ Add WHERE true if intentional

-- SELECT * in production code
SELECT * FROM large_table;  -- ❌ Specify columns

-- Sequential IDs for distribution
id SERIAL PRIMARY KEY  -- ❌ Creates hotspots

-- Large OFFSET pagination
SELECT * FROM table LIMIT 20 OFFSET 10000;  -- ❌ Inefficient
```

### DO Use These Instead
```sql
-- UUID primary keys
id UUID PRIMARY KEY DEFAULT gen_random_uuid()  -- ✅

-- Explicit PRIMARY KEY
CREATE TABLE good (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  data STRING
);  -- ✅

-- STRING type
description STRING  -- ✅

-- JSONB type
data JSONB  -- ✅

-- Timezone-aware timestamps
created_at TIMESTAMPTZ DEFAULT now()  -- ✅

-- Safe deletion
DELETE FROM table WHERE condition;  -- ✅
DELETE FROM table WHERE true;  -- ✅ Explicit full delete

-- Specific columns
SELECT id, name, email FROM users;  -- ✅

-- Keyset pagination
SELECT * FROM posts
WHERE (created_at, id) < ($1, $2)
ORDER BY created_at DESC, id DESC
LIMIT 20;  -- ✅
```

## Performance Best Practices

### 1. Index Strategy
- Create indexes for WHERE, JOIN, and ORDER BY columns
- Use STORING for frequently accessed columns
- Use partial indexes for filtered queries
- Use hash-sharded indexes for sequential data
- Monitor unused indexes and drop them

### 2. Query Patterns
- Use prepared statements for repeated queries
- Batch operations when possible
- Use RETURNING NOTHING when results aren't needed
- Prefer UPSERT over INSERT ON CONFLICT for blind writes
- Use appropriate transaction isolation levels

### 3. Data Distribution
- Use UUID primary keys for even distribution
- Avoid sequential IDs that cause hotspots
- Consider hash-sharded indexes for time-series data
- Split ranges manually for known access patterns
- Use zone configs for geo-distribution

### 4. Connection Management
- Use connection pooling
- Set appropriate statement timeout
- Use read replicas with AS OF SYSTEM TIME
- Consider follower reads for stale data tolerance

### 5. Monitoring
```sql
-- Check slow queries
SELECT * FROM crdb_internal.cluster_queries
WHERE start > now() - INTERVAL '5 minutes'
ORDER BY start DESC;

-- Check table statistics
SHOW STATISTICS FOR TABLE table_name;

-- Check index usage
SELECT * FROM crdb_internal.index_usage_statistics
WHERE table_name = 'your_table';

-- Explain query plan
EXPLAIN (VERBOSE) SELECT ...;
EXPLAIN ANALYZE SELECT ...;
```

## Query Plan Analysis

### Understanding EXPLAIN
```sql
-- Basic explain
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Explain with statistics
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Explain with all options
EXPLAIN (ANALYZE, VERBOSE, TYPES, DISTSQL)
SELECT * FROM users WHERE email = 'test@example.com';

-- Check for full table scans
EXPLAIN SELECT * FROM large_table;
-- Look for "full scan" in output
```

### Common Plan Issues
1. **Full table scans** - Add appropriate indexes
2. **Hash joins on large tables** - Consider merge joins
3. **High network latency** - Use zone configs
4. **Excessive round trips** - Batch operations
5. **Lock contention** - Reduce transaction scope

## Hardware and Deployment

### Resource Recommendations
- **CPU**: 4-8 cores per node minimum
- **Memory**: 8-16 GB per node minimum
- **Storage**: SSDs strongly recommended
- **Network**: Low latency between nodes

### Cluster Sizing
```sql
-- Check cluster capacity
SELECT * FROM crdb_internal.kv_node_status;

-- Check range distribution
SELECT * FROM crdb_internal.ranges;

-- Check replication status
SHOW RANGES FROM TABLE table_name;
```