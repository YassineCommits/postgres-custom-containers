# PostgreSQL 14 Extensions README

This document provides information about all the PostgreSQL extensions included in our custom PostgreSQL 14 Docker image.

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

**Description**: pglogical is a logical replication system for PostgreSQL, providing a more efficient and flexible way to replicate data compared to traditional PostgreSQL replication methods.

**Use Cases**:
- Multi-master replication
- Selective replication of specific tables or schemas
- Zero-downtime upgrades between PostgreSQL versions
- Cross-version replication

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pglogical;

-- Set up a provider (source)
SELECT pglogical.create_node(
    node_name := 'provider',
    dsn := 'host=localhost port=5432 dbname=postgres'
);

-- Set up a subscriber (target)
SELECT pglogical.create_subscription(
    subscription_name := 'subscription',
    provider_dsn := 'host=provider port=5432 dbname=postgres'
);
```

## pg_cron

**Description**: pg_cron enables PostgreSQL to schedule and execute jobs internally, similar to the Unix cron system but running inside the database.

**Use Cases**:
- Scheduling regular maintenance tasks (VACUUM, ANALYZE)
- Automating periodic data imports or exports
- Implementing data retention policies
- Running periodic calculations or aggregations

**Installation**: Already installed in the Docker image, but requires configuration.

**Configuration**:
Add to your postgresql.conf:
```
shared_preload_libraries = 'pg_cron'
cron.database_name = 'postgres'
```

**Usage**:
```sql
CREATE EXTENSION pg_cron;

-- Run a job every minute
SELECT cron.schedule('* * * * *', 'VACUUM ANALYZE mytable');

-- Run a job every day at midnight
SELECT cron.schedule('0 0 * * *', 'DELETE FROM events WHERE created_at < NOW() - INTERVAL ''90 days''');
```
## aiven-extras

**Description**: aiven-extras is a collection of PostgreSQL extensions and utilities developed by Aiven to enhance PostgreSQL functionality, particularly for cloud environments.

**Use Cases**:
- Provides additional monitoring tools
- Enhances cloud compatibility
- Improves operations in managed database services

**Installation**: Already included in the Docker image.

## PostGIS

**Description**: PostGIS adds support for geographic objects to PostgreSQL, allowing for location queries to be run in SQL.

**Use Cases**:
- Storing and querying spatial data (points, lines, polygons)
- Proximity analysis (finding nearest neighbors)
- Geofencing applications
- Routing and navigation systems
- Geographic data visualization

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION postgis;

-- Create a table with a geometry column
CREATE TABLE places (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    location GEOMETRY(Point, 4326)
);

-- Insert a point (longitude, latitude)
INSERT INTO places (name, location)
VALUES ('Eiffel Tower', ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326));

-- Find places within 5km of a point
SELECT name FROM places
WHERE ST_DWithin(
    location,
    ST_SetSRID(ST_MakePoint(2.3522, 48.8566), 4326),
    5000
);
```

## pg_hint_plan

**Description**: pg_hint_plan allows developers to control query execution plans using special hints in SQL comments, providing a way to override the query planner's decisions.

**Use Cases**:
- Fixing suboptimal query plans
- Performance tuning of complex queries
- Ensuring consistent execution plans in production
- Testing alternative execution strategies

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_hint_plan;

-- Force an index scan on the users table
/*+ IndexScan(users users_email_idx) */ 
SELECT * FROM users WHERE email = 'user@example.com';

-- Force a specific join order
/*+ Leading(orders customers) */
SELECT * FROM orders JOIN customers ON orders.customer_id = customers.id;
```

## pgrouting

**Description**: pgRouting extends PostGIS with geospatial routing and network analysis functionality.

**Use Cases**:
- Finding shortest paths in road networks
- Route planning and optimization
- Service area calculations
- Traveling salesperson problems
- Fleet management

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION postgis;
CREATE EXTENSION pgrouting;

-- Find the shortest path between two points in a road network
SELECT * FROM pgr_dijkstra(
    'SELECT id, source, target, cost, reverse_cost FROM road_network',
    5, 10, directed := true
);
```

## pg_partman

**Description**: pg_partman automates the creation and management of table partitions in PostgreSQL, handling both time-based and serial-based partitioning.

**Use Cases**:
- Managing large time-series data efficiently
- Automating partition creation, retention, and maintenance
- Improving query performance on partitionable data
- Simplifying data lifecycle management

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_partman;

-- Create a time-based partitioned table (daily partitions)
CREATE TABLE measurements (
    time TIMESTAMP NOT NULL,
    device_id INTEGER,
    value NUMERIC
) PARTITION BY RANGE (time);

-- Set up pg_partman to manage the partitions
SELECT partman.create_parent(
    p_parent_table := 'public.measurements',
    p_control := 'time',
    p_type := 'native',
    p_interval := 'daily',
    p_premake := 30
);

-- Set up background worker to automatically create future partitions
UPDATE partman.part_config
SET infinite_time_partitions = true,
    retention = '3 months',
    retention_keep_table = false
WHERE parent_table = 'public.measurements';
```

## wal2json

**Description**: wal2json is a logical decoding output plugin for PostgreSQL that converts WAL (Write-Ahead Log) entries to JSON format, making it easier to integrate with other systems.

**Use Cases**:
- Change data capture (CDC) pipelines
- Real-time data replication to non-PostgreSQL systems
- Event sourcing architectures
- Audit logging

**Installation**: Already included in the Docker image.

**Usage**:
```sql
-- Create a replication slot using wal2json
SELECT * FROM pg_create_logical_replication_slot('test_slot', 'wal2json');

-- Consume changes
SELECT * FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 'pretty-print', '1');
```

## ip4r

**Description**: ip4r provides IPv4 and IPv6 data types and operators for PostgreSQL, allowing for efficient storage and querying of IP addresses and networks.

**Use Cases**:
- IP address management systems
- Network configuration databases
- Security and access control applications
- Geolocation services based on IP ranges

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION ip4r;

-- Create a table with IP columns
CREATE TABLE access_logs (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    ip_address ip4r,
    request_path TEXT
);

-- Query for IP addresses in a specific subnet
SELECT * FROM access_logs 
WHERE ip_address <<= '192.168.1.0/24'::ip4r;
```

## pg_repack

**Description**: pg_repack allows for the reorganization of tables in PostgreSQL with minimal locks, reducing table and index bloat without blocking concurrent operations.

**Use Cases**:
- Eliminating table bloat in high-write environments
- Improving query performance on frequently updated tables
- Reclaiming disk space
- Maintenance operations with minimal downtime

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_repack;

-- Repack a bloated table
SELECT pg_repack.repack_table('mytable');

-- Repack all tables in the public schema
pg_repack -k -d mydb -s public
```

## decoderbufs

**Description**: decoderbufs is a logical decoding output plugin for PostgreSQL that formats WAL data as Protocol Buffers, which are efficient, language-neutral, extensible mechanisms for serializing structured data.

**Use Cases**:
- High-performance change data capture
- Integration with systems that use Protocol Buffers
- Streaming database changes to microservices

**Installation**: Already included in the Docker image.

**Usage**: Typically used with replication slots and external connectors like Debezium.

## orafce

**Description**: orafce provides Oracle-compatible functions for PostgreSQL, making it easier to migrate applications from Oracle to PostgreSQL.

**Use Cases**:
- Oracle to PostgreSQL migration projects
- Supporting applications with Oracle-specific SQL
- Maintaining compatibility with legacy code

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION orafce;

-- Use Oracle-compatible functions
SELECT nvl(null, 'default value');
SELECT to_char(sysdate, 'YYYY-MM-DD HH24:MI:SS');
```

## hypopg

**Description**: hypopg allows creating hypothetical indexes in PostgreSQL without actually creating the indexes on disk, enabling "what-if" analysis for index optimization.

**Use Cases**:
- Testing index strategies without modifying the database
- Performance tuning in production environments
- Index planning for large databases

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION hypopg;

-- Create a hypothetical index and see if it would be used
SELECT * FROM hypopg_create_index('CREATE INDEX ON orders(customer_id)');
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

-- Clean up
SELECT hypopg_reset();
```

## hll

**Description**: hll implements HyperLogLog, a probabilistic data structure for estimating the cardinality of large sets with a fixed small memory footprint.

**Use Cases**:
- Counting distinct values in very large datasets
- Real-time analytics with approximate counts
- Reducing memory usage for unique value counting
- Network monitoring and traffic analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION hll;

-- Create a table with an HLL column
CREATE TABLE user_visits (
    day DATE,
    page_id INTEGER,
    visitors hll
);

-- Add data to HLL
UPDATE user_visits 
SET visitors = hll_add(visitors, hll_hash_integer(user_id))
WHERE day = CURRENT_DATE AND page_id = 101;

-- Count distinct elements (with error < 2% typically)
SELECT hll_cardinality(visitors) FROM user_visits 
WHERE day = CURRENT_DATE AND page_id = 101;
```

## pg_qualstats

**Description**: pg_qualstats collects statistics about predicates in WHERE clauses and JOIN conditions, helping identify which columns would benefit most from indexing.

**Use Cases**:
- Index optimization
- Query performance troubleshooting
- Database monitoring and planning
- Automated index recommendations

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_qualstats;

-- Run some queries to collect data
SELECT * FROM users WHERE email = 'test@example.com';
SELECT * FROM orders WHERE customer_id = 123;

-- Analyze the collected statistics
SELECT * FROM pg_qualstats();

-- Find the most frequently filtered columns
SELECT relname, attname, opname, count, avg_sel
FROM pg_qualstats()
JOIN pg_class ON pg_qualstats.relid = pg_class.oid
JOIN pg_attribute ON pg_qualstats.attid = pg_attribute.attnum
                  AND pg_qualstats.relid = pg_attribute.attrelid
ORDER BY count DESC;
```

## pg_stat_kcache

**Description**: pg_stat_kcache gathers statistics about real CPU and physical disk access done by PostgreSQL, providing insights beyond what the built-in statistics collector offers.

**Use Cases**:
- Performance monitoring
- Resource usage analysis
- Identifying I/O bottlenecks
- Capacity planning

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_stat_kcache;

-- View system resource usage statistics
SELECT * FROM pg_stat_kcache();

-- Find the most CPU-intensive queries
SELECT query, calls, cpu_user, cpu_sys
FROM pg_stat_kcache JOIN pg_stat_statements USING (queryid)
ORDER BY (cpu_user + cpu_sys) DESC
LIMIT 10;
```

## PL/Python

**Description**: PL/Python allows writing PostgreSQL functions and procedures in Python, providing access to the Python ecosystem from within the database.

**Use Cases**:
- Complex data processing
- Machine learning integration
- API access from database functions
- Advanced text processing and data manipulation

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION plpython3u;

-- Create a Python function
CREATE FUNCTION pymax(a integer, b integer)
  RETURNS integer
AS $$
  return max(a, b)
$$ LANGUAGE plpython3u;

-- Create a function that uses external Python libraries
-- (requires installing the libraries in the container)
CREATE FUNCTION sentiment_analysis(text_to_analyze TEXT)
  RETURNS TEXT
AS $$
  import nltk
  from nltk.sentiment import SentimentIntensityAnalyzer
  
  sia = SentimentIntensityAnalyzer()
  sentiment = sia.polarity_scores(text_to_analyze)
  return 'positive' if sentiment['compound'] > 0.05 else 'negative' if sentiment['compound'] < -0.05 else 'neutral'
$$ LANGUAGE plpython3u;
```

## pg_freespacemap

**Description**: pg_freespacemap provides access to the free space map of tables and indexes, showing where and how much free space is available in each page.

**Use Cases**:
- Monitoring table bloat
- Understanding storage patterns
- Database maintenance planning
- Troubleshooting space usage issues

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_freespacemap;

-- View free space in a table
SELECT * FROM pg_freespace('mytable');

-- Analyze average free space percentage
SELECT count(*) AS pages,
       round(100 * avg(avail)::numeric / 8192, 2) AS avg_pct_free
FROM pg_freespace('mytable');
```

## pgAudit

**Description**: pgAudit provides detailed session and object audit logging through the standard PostgreSQL logging facility, meeting requirements for various security compliance standards.

**Use Cases**:
- Regulatory compliance (GDPR, HIPAA, SOX, etc.)
- Security auditing
- Activity monitoring
- Forensic analysis

**Installation**: Already installed in the Docker image, but requires configuration.

**Configuration**:
Add to your postgresql.conf:
```
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'write, ddl'
pgaudit.log_catalog = off
pgaudit.log_relation = on
pgaudit.log_statement_once = off
```

**Usage**:
```sql
CREATE EXTENSION pgaudit;

-- Set up role-based auditing
CREATE ROLE auditor;
GRANT auditor TO myuser;

-- Audit all SELECT statements for a user
ALTER ROLE myuser SET pgaudit.log TO 'read';
```

## pgVector

**Description**: pgVector adds vector similarity search capabilities to PostgreSQL, enabling efficient storage and querying of high-dimensional vector data commonly used in machine learning applications.

**Use Cases**:
- Similarity search for recommendation systems
- Image and text embeddings storage
- Machine learning feature vectors
- Natural language processing applications
- Nearest neighbor search

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION vector;

-- Create a table with vector data
CREATE TABLE items (
  id bigserial PRIMARY KEY,
  embedding vector(384)  -- 384 dimensional vector
);

-- Insert data
INSERT INTO items (embedding) VALUES ('[0.1, 0.2, ...]'::vector);

-- Create an index for faster similarity search
CREATE INDEX items_embedding_idx ON items USING ivfflat (embedding vector_l2_ops) WITH (lists = 100);

-- Find similar items (using L2 distance)
SELECT id, embedding <-> '[0.3, 0.4, ...]'::vector AS distance
FROM items
ORDER BY distance
LIMIT 5;
```

## pg_prewarm

**Description**: pg_prewarm allows loading relation data into either the operating system buffer cache or the PostgreSQL buffer cache.

**Use Cases**:
- Pre-warming cache after database restart
- Ensuring critical tables remain in memory
- Performance optimization for predictable workloads
- Benchmark testing with controlled cache states

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_prewarm;

-- Preload a table into the buffer cache
SELECT pg_prewarm('frequently_accessed_table');

-- Preload a specific index
SELECT pg_prewarm('my_important_index');

-- Specify buffer type: shared_buffers, operating system, or both
SELECT pg_prewarm('critical_table', 'buffer');
```

## PL/Perl

**Description**: PL/Perl allows writing PostgreSQL functions and procedures in Perl, bringing Perl's text processing capabilities into the database.

**Use Cases**:
- Advanced text processing and regular expressions
- Legacy code integration
- Complex string manipulation
- Data cleansing and validation

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION plperl;

-- Create a Perl function for regular expression processing
CREATE FUNCTION extract_phone_numbers(text)
RETURNS SETOF text
AS $$
  my $text = shift;
  my @phones = ();
  while ($text =~ /(\+\d{1,3}[\s-]?)?\(?\d{3}\)?[\s-]?\d{3}[\s-]?\d{4}/g) {
    push @phones, $&;
  }
  return \@phones;
$$ LANGUAGE plperl;

-- Example usage
SELECT extract_phone_numbers('Call me at (555) 123-4567 or +1 555-765-4321');
```

## tablefunc

**Description**: tablefunc provides various functions that return tables, especially useful for pivot operations (crosstab) and other advanced table transformations.

**Use Cases**:
- Creating pivot tables
- Time series analysis
- Converting row data to columnar format
- Custom reporting

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION tablefunc;

-- Create a simple crosstab pivot table
SELECT * FROM crosstab(
  'SELECT row_name, category, value 
   FROM source_table 
   ORDER BY 1,2',
  'SELECT DISTINCT category FROM source_table ORDER BY 1'
) AS ct(row_name text, category1 numeric, category2 numeric, category3 numeric);

-- Multi-column example (e.g., sales by region and quarter)
SELECT * FROM crosstab(
  'SELECT region, quarter, sum(sales) 
   FROM sales_data 
   GROUP BY 1, 2 
   ORDER BY 1, 2',
  'SELECT DISTINCT quarter FROM sales_data ORDER BY 1'
) AS ct(region text, q1 numeric, q2 numeric, q3 numeric, q4 numeric);
```

## btree_gin

**Description**: btree_gin allows B-tree equivalent behavior in GIN indexes for certain data types, enabling more efficient multi-column indexes for equality and range searches.

**Use Cases**:
- Improving multi-column index performance
- Combining full-text search with regular column filtering
- Better indexing for heterogeneous queries

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION btree_gin;

-- Create a GIN index with btree behavior
CREATE INDEX gin_multi_idx ON products USING gin (category, price);

-- Query that can use the index
SELECT * FROM products 
WHERE category = 'electronics' AND price BETWEEN 100 AND 500;
```

## btree_gist

**Description**: btree_gist enables GiST indexes to handle B-tree compatible operations, allowing the creation of exclusion constraints and combining different types of searches in a single index.

**Use Cases**:
- Creating exclusion constraints for time intervals
- Combining spatial and non-spatial data in queries
- Range overlap detection
- Scheduling and resource allocation applications

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION btree_gist;

-- Create a table with an exclusion constraint
CREATE TABLE room_reservations (
  room_id integer,
  reservation_time tsrange,
  EXCLUDE USING gist (room_id WITH =, reservation_time WITH &&)
);

-- This works fine
INSERT INTO room_reservations VALUES (1, '[2023-01-01 09:00, 2023-01-01 10:00)');

-- This fails due to the exclusion constraint (overlapping time)
INSERT INTO room_reservations VALUES (1, '[2023-01-01 09:30, 2023-01-01 10:30)');
```