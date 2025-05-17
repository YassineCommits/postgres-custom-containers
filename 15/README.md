# PostgreSQL 15 Extensions README

This document provides information about all the PostgreSQL extensions included in our custom PostgreSQL 15 Docker image.

## Table of Contents
- [aiven-extras](#aiven-extras)
- [pgVector](#pgvector)
- [pg_cron](#pg_cron)
- [PostGIS](#postgis)
- [pgAudit](#pgaudit)
- [orafce](#orafce)
- [pg_partman](#pg_partman)
- [pg_hint_plan](#pg_hint_plan)
- [wal2json](#wal2json)
- [postgres_fdw](#postgres_fdw)
- [tds_fdw](#tds_fdw)
- [hll](#hll)
- [PL/Python](#plpython)
- [PL/Perl](#plperl)
- [pgrouting](#pgrouting)
- [ip4r](#ip4r)
- [pg_similarity](#pg_similarity)
- [pg_stat_statements](#pg_stat_statements)

## aiven-extras

**Description**: aiven-extras is a collection of PostgreSQL extensions and utilities developed by Aiven to enhance PostgreSQL functionality, particularly for cloud environments.

**Use Cases**:
- Enhanced monitoring and diagnostics
- Cloud compatibility features
- Management utilities for hosted PostgreSQL
- Performance analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION aiven_extras;

-- Check version
SELECT aiven_extras_version();
```

## pgVector

**Description**: pgVector adds vector similarity search capabilities to PostgreSQL, enabling efficient storage and querying of embeddings and other vector data used in machine learning and AI applications.

**Use Cases**:
- Semantic search
- Recommendation systems
- Image similarity search
- Natural language processing applications
- ML feature vector storage and retrieval

**Installation**: Already included in the Docker image (version 0.8.0).

**Usage**:
```sql
CREATE EXTENSION vector;

-- Create a table with vector data
CREATE TABLE items (
  id bigserial PRIMARY KEY,
  embedding vector(1536)  -- OpenAI Ada embedding size
);

-- Insert data
INSERT INTO items (embedding) VALUES (
  '[0.1, 0.2, 0.3, ...]'::vector
);

-- Create an index for faster similarity search (HNSW index type added in v0.5.0)
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);

-- Find similar items (cosine similarity)
SELECT id, embedding <=> '[0.2, 0.3, 0.4, ...]'::vector AS similarity
FROM items
ORDER BY similarity
LIMIT 10;

-- L2 distance search (smaller is more similar)
SELECT id, embedding <-> '[0.2, 0.3, 0.4, ...]'::vector AS distance
FROM items
ORDER BY distance
LIMIT 10;

-- Inner product search (larger is more similar)
SELECT id, embedding <#> '[0.2, 0.3, 0.4, ...]'::vector AS product
FROM items
ORDER BY product DESC
LIMIT 10;
```

## pg_cron

**Description**: pg_cron is a job scheduler for PostgreSQL that runs inside the database as an extension, allowing you to schedule PostgreSQL commands directly from the database.

**Use Cases**:
- Scheduling regular maintenance tasks (VACUUM, ANALYZE)
- Implementing data retention policies
- Automated report generation
- Periodic data aggregation or ETL tasks

**Installation**: Already installed in the Docker image, but requires configuration.

**Configuration**:
Add to your postgresql.conf:
```
shared_preload_libraries = 'pg_cron'
cron.database_name = 'postgres'  # database where pg_cron runs
```

**Usage**:
```sql
CREATE EXTENSION pg_cron;

-- Run a job every minute
SELECT cron.schedule('* * * * *', 'VACUUM ANALYZE mytable');

-- Run a job every day at 3:30 AM
SELECT cron.schedule('30 3 * * *', 'DELETE FROM logs WHERE created_at < NOW() - INTERVAL ''30 days''');

-- Schedule a job with a custom name
SELECT cron.schedule('cleanup_old_data', '0 0 * * 0', 'CALL maintenance.cleanup_procedure()');

-- List all scheduled jobs
SELECT * FROM cron.job;

-- Delete a scheduled job
SELECT cron.unschedule('job_name');
```

## PostGIS

**Description**: PostGIS adds spatial database capabilities to PostgreSQL, allowing storage, retrieval, and analysis of geographic data.

**Use Cases**:
- GIS applications
- Location-based services
- Spatial data analysis
- Mapping and geospatial visualization
- Routing and navigation

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION postgis;

-- Create a table with geometry
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    location GEOMETRY(Point, 4326)  -- 4326 is the SRID for WGS84 (GPS coordinates)
);

-- Insert points (longitude, latitude)
INSERT INTO locations (name, location)
VALUES ('Eiffel Tower', ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326));

-- Spatial queries
-- Find locations within 5km of a point
SELECT name FROM locations
WHERE ST_DWithin(
    location,
    ST_SetSRID(ST_MakePoint(2.3522, 48.8566), 4326),  -- Paris coordinates
    5000  -- meters
);

-- Calculate distance between points
SELECT name, 
       ST_Distance(
           location::geography, 
           ST_SetSRID(ST_MakePoint(2.3522, 48.8566), 4326)::geography
       ) AS distance_meters
FROM locations
ORDER BY distance_meters;
```

## pgAudit

**Description**: pgAudit provides detailed session and object audit logging through PostgreSQL's standard logging facility, meeting requirements for various security compliance standards.

**Use Cases**:
- Regulatory compliance (GDPR, HIPAA, SOX, etc.)
- Security auditing and monitoring
- Activity tracking
- Access control validation

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

-- Audit a specific table for all operations
ALTER TABLE sensitive_data ENABLE AUDIT;
```

## orafce

**Description**: orafce (Oracle Foreign Compatibility Extensions) provides Oracle-compatible functions and packages for PostgreSQL, making it easier to migrate applications from Oracle to PostgreSQL.

**Use Cases**:
- Oracle to PostgreSQL migration projects
- Supporting applications with Oracle-specific SQL
- Maintaining compatibility with legacy systems

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION orafce;

-- Use Oracle-compatible functions
SELECT nvl(null, 'default value');
SELECT to_char(sysdate, 'YYYY-MM-DD HH24:MI:SS');

-- Use Oracle-compatible packages
SELECT dbms_random.value(1, 100);
```

## pg_partman

**Description**: pg_partman automates the creation and management of table partitions in PostgreSQL, supporting both time-based and serial-based partitioning strategies.

**Use Cases**:
- Managing large time-series data efficiently
- Automating partition creation and maintenance
- Improving query performance on large tables
- Implementing data retention policies

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_partman;

-- Create a time-based partitioned table
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

-- Create a background worker to automatically create future partitions
UPDATE partman.part_config
SET infinite_time_partitions = true,
    retention = '3 months',
    retention_keep_table = false
WHERE parent_table = 'public.measurements';

-- Create a serial-based partitioned table
CREATE TABLE orders (
    id BIGSERIAL NOT NULL,
    order_date TIMESTAMP,
    customer_id INTEGER
) PARTITION BY RANGE (id);

-- Set up pg_partman for ID-based partitioning
SELECT partman.create_parent(
    p_parent_table := 'public.orders',
    p_control := 'id',
    p_type := 'native',
    p_interval := '10000',
    p_premake := 5
);
```

## pg_hint_plan

**Description**: pg_hint_plan allows you to control query execution plans using special SQL comments called "hints," providing a way to override the query planner's decisions.

**Use Cases**:
- Performance tuning for complex queries
- Forcing specific plans when the optimizer makes suboptimal choices
- Testing different execution strategies
- Ensuring stable query plans in production

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_hint_plan;

-- Force an index scan on a table
/*+ IndexScan(users users_email_idx) */ 
SELECT * FROM users WHERE email = 'user@example.com';

-- Force a specific join method and order
/*+ Leading(orders customers) HashJoin(orders customers) */
SELECT * FROM orders o 
JOIN customers c ON o.customer_id = c.id
WHERE o.created_at > '2023-01-01';

-- Disable a specific index
/*+ NoIndexScan(users users_email_idx) */
SELECT * FROM users WHERE email = 'user@example.com';
```

## wal2json

**Description**: wal2json is a logical decoding output plugin for PostgreSQL that formats WAL (Write-Ahead Log) changes as JSON, making it easier to integrate with other systems.

**Use Cases**:
- Change data capture (CDC)
- Real-time data integration
- Event sourcing
- Replication to non-PostgreSQL systems

**Installation**: Already included in the Docker image.

**Usage**:
```sql
-- Create a replication slot using wal2json
SELECT * FROM pg_create_logical_replication_slot('test_slot', 'wal2json');

-- Make some changes
CREATE TABLE test_table (id serial primary key, data text);
INSERT INTO test_table (data) VALUES ('test data');
UPDATE test_table SET data = 'updated data' WHERE id = 1;

-- Consume the changes as JSON
SELECT * FROM pg_logical_slot_get_changes('test_slot', NULL, NULL);

-- With formatting options
SELECT * FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 
  'pretty-print', '1', 
  'include-timestamps', '1'
);

-- Clean up
SELECT pg_drop_replication_slot('test_slot');
```

## postgres_fdw

**Description**: postgres_fdw is a foreign data wrapper for PostgreSQL, allowing one PostgreSQL server to access data stored in other PostgreSQL servers as if they were local tables.

**Use Cases**:
- Distributed databases
- Database federation
- Data integration
- Cross-database queries
- Migration strategies

**Installation**: Already included in PostgreSQL 15 core.

**Usage**:
```sql
CREATE EXTENSION postgres_fdw;

-- Create a foreign server
CREATE SERVER foreign_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote-server.example.com', port '5432', dbname 'remote_db');

-- Create a user mapping
CREATE USER MAPPING FOR local_user
SERVER foreign_server
OPTIONS (user 'remote_user', password 'password');

-- Create a foreign table
CREATE FOREIGN TABLE remote_table (
    id integer,
    name text
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'original_table');

-- Query the foreign table like a local table
SELECT * FROM remote_table WHERE id > 1000;
```

## tds_fdw

**Description**: tds_fdw is a foreign data wrapper that allows PostgreSQL to connect to Microsoft SQL Server and Sybase databases.

**Use Cases**:
- Integration with Microsoft SQL Server
- Migration from SQL Server to PostgreSQL
- Cross-database reporting
- Heterogeneous database environments

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION tds_fdw;

-- Create a foreign server for SQL Server
CREATE SERVER sqlserver
FOREIGN DATA WRAPPER tds_fdw
OPTIONS (servername 'sql-server.example.com', port '1433', database 'Northwind');

-- Create a user mapping
CREATE USER MAPPING FOR postgres
SERVER sqlserver
OPTIONS (username 'sa', password 'password');

-- Create a foreign table
CREATE FOREIGN TABLE customers (
    CustomerID char(5),
    CompanyName varchar(40),
    ContactName varchar(30),
    Country varchar(15)
)
SERVER sqlserver
OPTIONS (schema_name 'dbo', table_name 'Customers');

-- Query the SQL Server table from PostgreSQL
SELECT * FROM customers WHERE Country = 'Germany';
```

## hll

**Description**: hll (HyperLogLog) is a fixed-size data structure for probabilistic counting of distinct values, requiring significantly less memory than exact counting methods.

**Use Cases**:
- Approximate unique value counts in large datasets
- Web analytics (unique visitors)
- Real-time metrics with low memory usage
- Large-scale data analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION hll;

-- Create a table with an HLL column
CREATE TABLE page_views (
    day DATE,
    page_id INTEGER,
    visitor_hll hll
);

-- Insert data by hashing user IDs
INSERT INTO page_views 
VALUES (CURRENT_DATE, 1, hll_empty() || hll_hash_integer(101));

-- Add more user IDs to the HLL
UPDATE page_views 
SET visitor_hll = visitor_hll || hll_hash_integer(102) || hll_hash_integer(103)
WHERE day = CURRENT_DATE AND page_id = 1;

-- Count approximate distinct users
SELECT hll_cardinality(visitor_hll) FROM page_views
WHERE day = CURRENT_DATE AND page_id = 1;

-- Combine HLLs from different pages
SELECT SUM(hll_cardinality(visitor_hll)) AS total_page_views,
       hll_cardinality(hll_union_agg(visitor_hll)) AS unique_visitors
FROM page_views
WHERE day >= CURRENT_DATE - INTERVAL '7 days';
```

## PL/Python

**Description**: PL/Python allows you to write PostgreSQL functions and procedures in Python, bringing the power and flexibility of Python to database functions.

**Use Cases**:
- Complex data processing
- Machine learning and AI integration
- API calls within database functions
- Text and data analysis
- Custom business logic

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION plpython3u;

-- Create a simple Python function
CREATE FUNCTION pymax(a integer, b integer)
  RETURNS integer
AS $$
  return max(a, b)
$$ LANGUAGE plpython3u;

-- Function with Python libraries (must be installed in the container)
CREATE FUNCTION sentiment(text_content text)
  RETURNS text
AS $$
  # Requires 'pip install textblob' in the container
  from textblob import TextBlob
  
  analysis = TextBlob(text_content)
  polarity = analysis.sentiment.polarity
  
  if polarity > 0.1:
      return "positive"
  elif polarity < -0.1:
      return "negative"
  else:
      return "neutral"
$$ LANGUAGE plpython3u;

-- Function that returns a set
CREATE FUNCTION py_word_frequencies(document text)
  RETURNS TABLE (word text, frequency integer)
AS $$
  from collections import Counter
  import re
  
  words = re.findall(r'\w+', document.lower())
  word_counts = Counter(words).most_common(10)
  
  return [(word, count) for word, count in word_counts]
$$ LANGUAGE plpython3u;
```

## PL/Perl

**Description**: PL/Perl allows you to write PostgreSQL functions and procedures in Perl, bringing Perl's powerful text processing capabilities to the database.

**Use Cases**:
- Complex text processing and regular expressions
- Legacy code integration
- Data cleansing and validation
- Log parsing and analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION plperl;

-- Create a simple Perl function
CREATE FUNCTION perl_max(integer, integer) RETURNS integer AS $$
    my ($a, $b) = @_;
    return $a > $b ? $a : $b;
$$ LANGUAGE plperl;

-- Function with regular expressions
CREATE FUNCTION extract_emails(text) RETURNS SETOF text AS $$
    my $text = shift;
    my @emails = ();
    while ($text =~ /([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})/g) {
        push @emails, $1;
    }
    return \@emails;
$$ LANGUAGE plperl;

-- Function that modifies input and returns a record
CREATE TYPE person AS (name text, age integer);

CREATE FUNCTION format_person(text) RETURNS person AS $$
    my $input = shift;
    my ($name, $age) = split /,/, $input;
    $name = ucfirst(lc($name));  # Capitalize first letter
    return { name => $name, age => $age };
$$ LANGUAGE plperl;
```

## pgrouting

**Description**: pgRouting extends PostGIS to provide geospatial routing and network analysis functionality for PostgreSQL.

**Use Cases**:
- Shortest path calculations
- Vehicle routing problems
- Service area analysis
- Traveling salesperson problems
- Network topology analysis

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION postgis;
CREATE EXTENSION pgrouting;

-- Create a table for road network
CREATE TABLE road_network (
    id BIGSERIAL PRIMARY KEY,
    source BIGINT,
    target BIGINT,
    cost FLOAT,
    reverse_cost FLOAT,
    geom GEOMETRY(LINESTRING, 4326)
);

-- Populate the table with road data
-- ...

-- Find the shortest path between two points
SELECT * FROM pgr_dijkstra(
    'SELECT id, source, target, cost, reverse_cost FROM road_network',
    5, 10, -- source and target vertices
    directed := true
);

-- Calculate a driving route and return the geometry
SELECT seq, node, edge, cost, agg_cost,
       ST_AsText(geom) AS path_geometry
FROM pgr_dijkstra(
    'SELECT id, source, target, cost, reverse_cost FROM road_network',
    5, 10, directed := true
) AS di
JOIN road_network rn ON di.edge = rn.id;
```

## ip4r

**Description**: ip4r provides IPv4 and IPv6 address types and operators for PostgreSQL, allowing efficient storage and querying of IP addresses and networks.

**Use Cases**:
- IP address management
- Network inventory systems
- Firewall rule management
- Security and access control
- Geolocation based on IP ranges

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION ip4r;

-- Create a table with IP columns
CREATE TABLE access_logs (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    ip_address ip4r, -- Can store IPv4 or IPv6
    request_path TEXT
);

-- Insert IPv4 addresses
INSERT INTO access_logs (ip_address, request_path)
VALUES ('192.168.1.1', '/index.html');

-- Insert IPv6 addresses
INSERT INTO access_logs (ip_address, request_path)
VALUES ('2001:db8::1', '/about.html');

-- Query for IP addresses in a specific subnet
SELECT * FROM access_logs 
WHERE ip_address <<= '192.168.1.0/24'; -- contained within subnet

-- Check if an IP is in multiple networks
SELECT * FROM access_logs 
WHERE ip_address << '10.0.0.0/8' OR ip_address << '172.16.0.0/12';
```

## pg_similarity

**Description**: pg_similarity provides functions and operators to determine the similarity between two strings, enabling fuzzy string matching and similarity searches.

**Use Cases**:
- Fuzzy text search
- Duplicate detection
- Record linkage and data deduplication
- Typo correction
- Name matching

**Installation**: Already included in the Docker image.

**Usage**:
```sql
CREATE EXTENSION pg_similarity;

-- Set default similarity measure
SELECT set_similarity_threshold('cosine', 0.7);

-- Create a table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT
);

INSERT INTO products (name) VALUES 
    ('iPhone 13 Pro Max'),
    ('iPhone 13 Pro'),
    ('Samsung Galaxy S21'),
    ('Samsung Galaxy Note');

-- Find similar product names using different algorithms
SELECT a.name, b.name, 
       similarity_cosine(a.name, b.name) AS cosine,
       similarity_levenshtein(a.name, b.name) AS levenshtein,
       similarity_jaro_winkler(a.name, b.name) AS jaro_winkler
FROM products a, products b
WHERE a.id < b.id
  AND a.name % b.name  -- Using the default similarity operator
ORDER BY cosine DESC;

-- Find products similar to a search term
SELECT name, similarity_cosine('iphone pro', name) AS similarity
FROM products
WHERE 'iphone pro' % name
ORDER BY similarity DESC;
```

## pg_stat_statements

**Description**: pg_stat_statements tracks execution statistics for all SQL statements executed by the server, providing insights into query performance and resource usage.

**Use Cases**:
- Query performance monitoring
- Identifying slow or resource-intensive queries
- Database optimization
- Troubleshooting performance issues
- Capacity planning

**Installation**: Included in PostgreSQL core, but needs to be enabled.

**Configuration**:
Add to your postgresql.conf:
```
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all
```

**Usage**:
```sql
CREATE EXTENSION pg_stat_statements;

-- View the most time-consuming queries
SELECT query, calls, total_exec_time, rows, mean_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Find queries with the highest average execution time
SELECT query, calls, mean_exec_time, stddev_exec_time
FROM pg_stat_statements
WHERE calls > 100  -- Only consider frequently called queries
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Identify queries reading the most blocks
SELECT query, shared_blks_read + local_blks_read AS total_blocks_read,
       calls,
       (shared_blks_read + local_blks_read) / calls AS avg_blocks_per_call
FROM pg_stat_statements
ORDER BY total_blocks_read DESC
LIMIT 10;

-- Reset statistics (e.g., after tuning)
SELECT pg_stat_statements_reset();
```