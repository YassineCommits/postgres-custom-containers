# PostgreSQL Extensions README

This document provides an overview of all the extensions included in the custom PostgreSQL image defined in the Dockerfile.

## Table of Contents
- [Core Functionality Extensions](#core-functionality-extensions)
- [Data Type Extensions](#data-type-extensions)
- [Analytics and Performance Extensions](#analytics-and-performance-extensions)
- [Replication and Auditing Extensions](#replication-and-auditing-extensions)
- [Foreign Data Wrapper Extensions](#foreign-data-wrapper-extensions)
- [Procedural Language Extensions](#procedural-language-extensions)
- [Geographic and Spatial Extensions](#geographic-and-spatial-extensions)
- [Management and Scheduling Extensions](#management-and-scheduling-extensions)

## Core Functionality Extensions

### aiven-extras
**Description**: A collection of utility functions and extensions developed by Aiven to enhance PostgreSQL functionality.  
**Use cases**: Database maintenance tasks, monitoring, and additional utilities for managing PostgreSQL instances.  
**Source**: [GitHub - aiven/aiven-extras](https://github.com/aiven/aiven-extras)

### orafce
**Description**: Oracle compatibility functions for PostgreSQL, providing Oracle-like functionality.  
**Use cases**: Migration from Oracle to PostgreSQL, maintaining compatibility with Oracle-specific code.  
**Source**: [GitHub - orafce/orafce](https://github.com/orafce/orafce)

## Data Type Extensions

### pgvector
**Description**: Open-source vector similarity search for PostgreSQL, enabling vector storage and similarity search operations.  
**Use cases**: Machine learning applications, recommendation systems, image similarity search, and other AI applications.  
**Version**: v0.8.0  
**Source**: [GitHub - pgvector/pgvector](https://github.com/pgvector/pgvector)

### ip4r
**Description**: IPv4/IPv6 data types for PostgreSQL, providing native support for IP address data.  
**Use cases**: Network applications, IP-based geolocation, and firewall rule management.  
**Source**: [GitHub - RhodiumToad/ip4r](https://github.com/RhodiumToad/ip4r)

### hll (HyperLogLog)
**Description**: Type for storing HyperLogLog data structures, which allow for efficient cardinality estimation.  
**Use cases**: Analyzing large datasets when approximate distinct-value counts are sufficient, reducing memory and computational requirements.  
**Source**: [GitHub - citusdata/postgresql-hll](https://github.com/citusdata/postgresql-hll)

## Analytics and Performance Extensions

### pg_hint_plan
**Description**: Controls query execution plans by providing hints to the planner, similar to Oracle's hint functionality.  
**Use cases**: Performance tuning for complex queries, ensuring consistent query plans.  
**Version**: REL16_1_6_0  
**Source**: [GitHub - ossc-db/pg_hint_plan](https://github.com/ossc-db/pg_hint_plan)

### pg_stat_statements
**Description**: Tracks execution statistics for all SQL statements executed by the server.  
**Use cases**: Query performance analysis, identifying slow queries, and database optimization.  
**Note**: Core PostgreSQL extension (included in shared_preload_libraries)

## Replication and Auditing Extensions

### pglogical
**Description**: Provides logical replication support for PostgreSQL.  
**Use cases**: Cross-version replication, selective replication, multi-master replication setups.  
**Source**: Available in PostgreSQL repositories (postgresql-16-pglogical)

### pgAudit
**Description**: Provides detailed session and object audit logging via the standard PostgreSQL logging facility.  
**Use cases**: Compliance requirements, security auditing, tracking database changes.  
**Version**: REL_16_STABLE  
**Source**: [GitHub - pgaudit/pgaudit](https://github.com/pgaudit/pgaudit)

### wal2json
**Description**: Output plugin for logical decoding that produces JSON format.  
**Use cases**: Change data capture (CDC), streaming database changes to other systems in JSON format.  
**Source**: [GitHub - eulerto/wal2json](https://github.com/eulerto/wal2json)

## Foreign Data Wrapper Extensions

### postgres_fdw
**Description**: Foreign data wrapper for accessing data stored in external PostgreSQL servers.  
**Use cases**: Federated database setups, distributed queries across multiple PostgreSQL instances.  
**Note**: Core PostgreSQL extension (requires CREATE EXTENSION)

### tds_fdw
**Description**: Foreign data wrapper for accessing Microsoft SQL Server and Sybase databases.  
**Use cases**: Integration with Microsoft SQL Server, data migration, cross-database reporting.  
**Source**: [GitHub - tds-fdw/tds_fdw](https://github.com/tds-fdw/tds_fdw)

## Procedural Language Extensions

### PL/Python
**Description**: Allows writing PostgreSQL functions in Python.  
**Use cases**: Complex data processing, integration with Python libraries, implementing business logic in Python.  
**Source**: Available in PostgreSQL repositories (postgresql-plpython3-16)

### PL/Perl
**Description**: Allows writing PostgreSQL functions in Perl.  
**Use cases**: Text processing, regular expressions, legacy code compatibility.  
**Source**: Available in PostgreSQL repositories (postgresql-plperl-16)

### PL/v8
**Description**: Procedural language for PostgreSQL powered by V8 JavaScript Engine.  
**Use cases**: JavaScript/JSON data processing, web application integration.  
**Source**: Available in PostgreSQL repositories or [GitHub - plv8/plv8](https://github.com/plv8/plv8)

## Geographic and Spatial Extensions

### PostGIS
**Description**: Spatial and geographic objects for PostgreSQL, adding support for geographic objects and queries.  
**Use cases**: GIS applications, location-based services, spatial analysis, mapping applications.  
**Source**: Available in PostgreSQL repositories (postgresql-16-postgis-3)

### pgRouting
**Description**: Extends PostGIS with geospatial routing functionality.  
**Use cases**: Shortest path analysis, route planning, logistics optimization.  
**Source**: [GitHub - pgRouting/pgrouting](https://github.com/pgRouting/pgrouting)

## Management and Scheduling Extensions

### pg_cron
**Description**: Job scheduler for PostgreSQL, allowing scheduling of PostgreSQL commands directly from the database.  
**Use cases**: Database maintenance, scheduled data processing, automated reporting.  
**Version**: v1.6.3  
**Source**: [GitHub - citusdata/pg_cron](https://github.com/citusdata/pg_cron)

### pg_partman
**Description**: Extension for creating and managing time-based or serial-based table partitions.  
**Use cases**: High-volume data management, time-series data, improving query performance on large tables.  
**Version**: v4.7.3  
**Source**: [GitHub - pgpartman/pg_partman](https://github.com/pgpartman/pg_partman)

## Configuration Note

Some extensions require adding them to the `shared_preload_libraries` parameter in your PostgreSQL configuration. The following should be added to your `postgresql.conf`:

```
shared_preload_libraries = 'pg_stat_statements,pg_cron,pgaudit'
```

After starting PostgreSQL, you can enable an extension with:

```sql
CREATE EXTENSION extension_name;
```

Replace `extension_name` with the name of the extension you want to enable.