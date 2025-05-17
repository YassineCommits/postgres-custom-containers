# PostgreSQL Extensions README

This document provides information about all the PostgreSQL extensions included in our custom PostgreSQL 13 Docker image.

## Table of Contents
- [pglogical](#pglogical)
- [pg_cron](#pg_cron)
- [aiven-extras](#aiven-extras)
- [PostGIS](#postgis)
- [pg_hint_plan](#pg_hint_plan)
- [pgrouting](#pgrouting)
- [pg_partman](#pg_partman)
- [wal2json](#wal2json)
- [ip4r](#ip4r)
- [pg_repack](#pg_repack)
- [decoderbufs](#decoderbufs)
- [orafce](#orafce)
- [hypopg](#hypopg)
- [hll](#hll)
- [pg_qualstats](#pg_qualstats)
- [pg_stat_kcache](#pg_stat_kcache)
- [PL/Python](#plpython)
- [pg_freespacemap](#pg_freespacemap)
- [pgAudit](#pgaudit)
- [pgVector](#pgvector)
- [pg_prewarm](#pg_prewarm)
- [PL/Perl](#plperl)
- [tablefunc](#tablefunc)
- [btree_gin](#btree_gin)
- [btree_gist](#btree_gist)

## pglogical

**Description**: pglogical is a logical replication system for PostgreSQL, designed for highly efficient and flexible replication. It provides a more powerful alternative to PostgreSQL's built-in replication features.

**Use Cases**:
- Multi-master replication
- Real-time data integration between multiple databases
- Data migration with minimal downtime

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pglogical;
```

## pg_cron

**Description**: pg_cron is a simple cron-based job scheduler for PostgreSQL that runs inside the database as an extension. It allows you to schedule PostgreSQL commands directly from the database.

**Use Cases**:
- Scheduling regular database maintenance tasks
- Automating periodic data processing
- Scheduling regular backups

**Installation**: Already installed in the Docker image, but requires configuration in postgresql.conf.

**Configuration**:
Add to your postgresql.conf:
```
shared_preload_libraries = 'pg_cron'
cron.database_name = 'postgres'
```

**Usage**:
```sql
CREATE EXTENSION pg_cron;

-- Schedule a job to run every minute
SELECT cron.schedule('* * * * *', 'VACUUM ANALYZE mytable');
```

## aiven-extras

**Description**: aiven-extras is a collection of PostgreSQL extensions and utilities developed by Aiven to enhance PostgreSQL functionality, particularly for cloud environments.

**Use Cases**:
- Provides additional monitoring tools
- Enhances cloud compatibility
- Improves operations in managed database services

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION aiven_extras;
```

## PostGIS

**Description**: PostGIS is a spatial database extender for PostgreSQL, adding support for geographic objects and location queries.

**Use Cases**:
- GIS (Geographic Information Systems) applications
- Location-based services
- Spatial analysis and mapping
- Geocoding and routing

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION postgis;
```

## pg_hint_plan

**Description**: pg_hint_plan allows you to control query execution plans using special SQL comments called "hints." This gives you more control over the query optimizer.

**Use Cases**:
- Troubleshooting poor query performance
- Forcing specific execution plans when the optimizer makes suboptimal choices
- Query optimization in complex applications

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_hint_plan;

-- Using a hint to force a specific plan
/*+ SeqScan(users) */ SELECT * FROM users WHERE id < 100;
```

## pgrouting

**Description**: pgRouting extends PostGIS to provide geospatial routing functionality.

**Use Cases**:
- Shortest path calculations
- Travel time calculations
- Routing applications for transportation
- Fleet management and logistics

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION postgis;
CREATE EXTENSION pgrouting;
```

## pg_partman

**Description**: pg_partman is an extension to create and manage time-based and serial-based table partitions.

**Use Cases**:
- Automating partition creation and management
- Improving performance of large time-series datasets
- Managing historical data efficiently

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_partman;

-- Create a time-based partition
SELECT partman.create_parent('public.measurement', 'time', 'native', 'daily');
```

## wal2json

**Description**: wal2json is a logical decoding output plugin for PostgreSQL that produces JSON output from WAL (Write-Ahead Logging).

**Use Cases**:
- Change data capture (CDC)
- Event sourcing
- Replication to non-PostgreSQL systems
- Real-time data integration

**Installation**: Already included in the Docker image.

**Usage**: Requires configuration with PostgreSQL's logical replication slots.

## ip4r

**Description**: ip4r is an extension that provides IPv4 and IPv6 address types and operators for PostgreSQL.

**Use Cases**:
- IP address management
- Network analysis
- Security applications
- Geolocation services

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION ip4r;

-- Create a table with IP address columns
CREATE TABLE network_events (
    id serial PRIMARY KEY,
    ip_address ip4r
);
```

## pg_repack

**Description**: pg_repack is an extension to remove bloat from tables and indexes without needing a heavy exclusive lock.

**Use Cases**:
- Database maintenance
- Reclaiming disk space
- Improving performance by reducing table bloat
- Reorganizing tables with minimal downtime

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_repack;

-- Repack a bloated table
SELECT pg_repack.repack_table('schema.table');
```

## decoderbufs

**Description**: decoderbufs is a logical decoding output plugin for PostgreSQL that produces Protocol Buffers messages from WAL.

**Use Cases**:
- Change data capture for high-performance applications
- Integration with systems that support Protocol Buffers
- Real-time data pipelines

**Installation**: Already included in the Docker image.

**Usage**: Requires configuration with PostgreSQL's logical replication slots.

## orafce

**Description**: orafce is an extension that provides Oracle-compatible functions for PostgreSQL, making it easier to migrate from Oracle to PostgreSQL.

**Use Cases**:
- Oracle to PostgreSQL migration
- Cross-database compatibility
- Supporting applications originally designed for Oracle

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION orafce;
```

## hypopg

**Description**: hypopg allows you to create hypothetical indexes without actually creating them on disk, letting you test index performance without affecting the database.

**Use Cases**:
- Index planning and testing
- Query optimization without modifying the database
- Performance analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION hypopg;

-- Create a hypothetical index
SELECT hypopg_create_index('CREATE INDEX ON orders(customer_id)');
```

## hll

**Description**: hll implements HyperLogLog, a fixed-size, set-like structure that allows for efficient cardinality estimation of datasets with massive unique counts.

**Use Cases**:
- Estimating unique visitor counts
- Analyzing large datasets with limited memory
- Cardinality estimation for big data

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION hll;

-- Use HLL for counting unique values
SELECT hll_add_agg(hll_hash_text(user_id)) FROM visits;
```

## pg_qualstats

**Description**: pg_qualstats is an extension to track query predicate statistics, helping identify which columns would benefit most from indexing.

**Use Cases**:
- Index optimization
- Query performance troubleshooting
- Database monitoring

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_qualstats;

-- View predicate statistics
SELECT * FROM pg_qualstats();
```

## pg_stat_kcache

**Description**: pg_stat_kcache gathers statistics about real CPU and disk usage in PostgreSQL.

**Use Cases**:
- Performance monitoring
- Resource usage analysis
- Identifying bottlenecks in system performance

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_stat_kcache;

-- View system resource usage statistics
SELECT * FROM pg_stat_kcache();
```

## PL/Python

**Description**: PL/Python allows you to write PostgreSQL functions in Python.

**Use Cases**:
- Complex data processing
- Machine learning integration
- API integrations
- Data transformations that are difficult in SQL

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION plpython3u;

-- Create a Python function
CREATE FUNCTION pymax(a integer, b integer)
  RETURNS integer
AS $$
  if a > b:
    return a
  return b
$$ LANGUAGE plpython3u;
```

## pg_freespacemap

**Description**: pg_freespacemap provides access to the free space map (FSM) data, showing available space in tables and indexes.

**Use Cases**:
- Database maintenance planning
- Monitoring table bloat
- Investigating space usage patterns

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_freespacemap;

-- View free space in a table
SELECT * FROM pg_freespace('mytable');
```

## pgAudit

**Description**: pgAudit provides detailed session and object audit logging through PostgreSQL's standard logging facility.

**Use Cases**:
- Compliance with security regulations (GDPR, HIPAA, etc.)
- Database activity monitoring
- Security auditing

**Installation**: Already installed in the Docker image, but requires configuration in postgresql.conf.

**Configuration**:
Add to your postgresql.conf:
```
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'write, ddl'
```

**Usage**:
```sql
CREATE EXTENSION pgaudit;
```

## pgVector

**Description**: pgVector adds vector similarity search capabilities to PostgreSQL, supporting embeddings and vector operations.

**Use Cases**:
- Machine learning applications
- Natural language processing
- Image similarity search
- Recommendation systems
- Embeddings storage and retrieval

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION vector;

-- Create a table with a vector column
CREATE TABLE items (
  id bigserial PRIMARY KEY,
  embedding vector(384)
);

-- Find similar items
SELECT * FROM items ORDER BY embedding <-> '[0.1, 0.2, ...]' LIMIT 5;
```

## pg_prewarm

**Description**: pg_prewarm allows you to preload relation data into the operating system buffer cache or PostgreSQL buffer cache.

**Use Cases**:
- Improving performance after database restarts
- Ensuring critical tables are cached
- Benchmark testing with controlled cache states

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_prewarm;

-- Prewarm a table
SELECT pg_prewarm('mytable');
```

## PL/Perl

**Description**: PL/Perl allows you to write PostgreSQL functions in Perl.

**Use Cases**:
- Text processing and regular expressions
- Legacy code integration
- Complex data transformations

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION plperl;

-- Create a Perl function
CREATE FUNCTION perl_max(integer, integer) RETURNS integer AS $$
    my ($a, $b) = @_;
    return $a > $b ? $a : $b;
$$ LANGUAGE plperl;
```

## tablefunc

**Description**: tablefunc provides various functions that return tables, particularly for crosstab (pivot table) operations.

**Use Cases**:
- Creating pivot tables
- Converting row-based data to column-based data
- Reporting and data analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION tablefunc;

-- Create a crosstab (pivot table)
SELECT * FROM crosstab(
  'SELECT row_name, category, value FROM source_table ORDER BY 1,2',
  'SELECT DISTINCT category FROM source_table ORDER BY 1'
) AS ct (row_name text, category1 numeric, category2 numeric, category3 numeric);
```

## btree_gin

**Description**: btree_gin allows B-tree equivalent behavior for GIN indexes for certain data types.

**Use Cases**:
- Improving performance for equality and range searches in multi-column indexes
- Combining full-text search with standard column filtering

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION btree_gin;

-- Create a GIN index with btree behavior
CREATE INDEX gin_idx ON table USING gin (column1, column2);
```

## btree_gist

**Description**: btree_gist provides GiST index operator classes with B-tree behavior for certain data types.

**Use Cases**:
- Creating exclusion constraints
- Combining spatial and non-spatial filtering
- Range queries with additional constraints

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION btree_gist;

-- Create an exclusion constraint using GiST
CREATE TABLE reservations (
  room int,
  during tsrange,
  EXCLUDE USING gist (room WITH =, during WITH &&)
);
```