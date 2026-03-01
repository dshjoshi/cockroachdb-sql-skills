## Dropping Schema Objects

### Drop Table
```sql
-- Basic drop
DROP TABLE users;

-- Safe drop (no error if missing)
DROP TABLE IF EXISTS users;

-- Drop with all dependents (foreign keys, views, etc.)
DROP TABLE users CASCADE;

-- Drop multiple tables
DROP TABLE IF EXISTS users, orders, products CASCADE;

-- Note: Cannot drop and recreate a table with the same name
-- in the same transaction. Split into separate transactions.
```

### Drop View
```sql
-- Drop regular view
DROP VIEW IF EXISTS user_summary CASCADE;

-- Drop materialized view (separate statement required)
DROP MATERIALIZED VIEW IF EXISTS monthly_stats CASCADE;

-- Note: DROP VIEW on a materialized view will fail.
-- Use the correct statement for each type.
```

### Drop Index
```sql
-- CockroachDB-specific table@index syntax (preferred)
DROP INDEX users@idx_email;

-- With IF EXISTS
DROP INDEX IF EXISTS users@idx_email;

-- Fully qualified
DROP INDEX mydb.public.users@idx_email;

-- CASCADE required for inline UNIQUE indexes (defined in CREATE TABLE)
-- UNIQUE indexes created via CREATE INDEX do not require CASCADE
DROP INDEX users@idx_unique_email CASCADE;

-- CASCADE required for hash-sharded indexes (drops shard column, triggers table rewrite)
DROP INDEX events@idx_sharded CASCADE;

-- Drop multiple indexes
DROP INDEX users@idx_email, orders@idx_user CASCADE;

-- CONCURRENTLY is a no-op (all CockroachDB schema changes are online)
DROP INDEX CONCURRENTLY users@idx_email;
```

