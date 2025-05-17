FROM postgres:17

# Install essential build dependencies and common packages
RUN apt-get update \
    && apt-get install -y make gcc git-all postgresql-server-dev-17 postgresql-17-pglogical postgresql-17-cron \
    libkrb5-dev libssl-dev zlib1g-dev libreadline-dev libxml2-dev libxslt1-dev libpq-dev \
    wget unzip curl build-essential libedit-dev libpam0g-dev libpython3-dev uuid-dev

# Install aiven-extras extension
RUN git clone https://github.com/aiven/aiven-extras.git /tmp/aiven-extras \
    && cd /tmp/aiven-extras \
    && make \
    && make install \
    && rm -rf /tmp/aiven-extras

# Install pgvector extension
RUN git clone --branch v0.8.0 https://github.com/pgvector/pgvector.git /tmp/pgvector \
    && cd /tmp/pgvector \
    && make \
    && make install \
    && rm -rf /tmp/pgvector

# Install pg_cron extension
RUN git clone --branch v1.6.3 https://github.com/citusdata/pg_cron.git /tmp/pg_cron \
    && cd /tmp/pg_cron \
    && make \
    && make install \
    && rm -rf /tmp/pg_cron

# Install PostGIS extension
RUN apt-get install -y postgresql-17-postgis-3

# Install pgAudit
RUN git clone https://github.com/pgaudit/pgaudit.git /tmp/pgaudit \
    && cd /tmp/pgaudit \
    && git checkout REL_17_STABLE \
    && make USE_PGXS=1 PG_CONFIG=/usr/bin/pg_config \
    && make install USE_PGXS=1 PG_CONFIG=/usr/bin/pg_config \
    && rm -rf /tmp/pgaudit


# Install orafce extension
RUN apt-get install -y postgresql-17-orafce || \
    ( git clone https://github.com/orafce/orafce.git /tmp/orafce \
    && cd /tmp/orafce \
    && make USE_PGXS=1 \
    && make USE_PGXS=1 install \
    && rm -rf /tmp/orafce )

# Install pg_partman extension
RUN git clone https://github.com/pgpartman/pg_partman.git /tmp/pg_partman \
    && cd /tmp/pg_partman \
    && git checkout v4.7.3 \
    && make \
    && make install \
    && rm -rf /tmp/pg_partman

# Install pg_hint_plan extension
RUN git clone https://github.com/ossc-db/pg_hint_plan.git /tmp/pg_hint_plan \
    && cd /tmp/pg_hint_plan \
    && git checkout REL16_1_6_0 \
    && make \
    && make install \
    && rm -rf /tmp/pg_hint_plan

# Install wal2json extension
RUN git clone https://github.com/eulerto/wal2json.git /tmp/wal2json \
    && cd /tmp/wal2json \
    && make \
    && make install \
    && rm -rf /tmp/wal2json

# Install postgres_fdw extension (included in PostgreSQL 17 core)
# Just needs to be enabled with CREATE EXTENSION



# Install tds_fdw extension
RUN apt-get install -y freetds-dev \
    && git clone https://github.com/tds-fdw/tds_fdw.git /tmp/tds_fdw \
    && cd /tmp/tds_fdw \
    && make USE_PGXS=1 \
    && make USE_PGXS=1 install \
    && rm -rf /tmp/tds_fdw

# Install hll extension
RUN apt-get install -y postgresql-17-hll || \
    ( git clone https://github.com/citusdata/postgresql-hll.git /tmp/hll \
    && cd /tmp/hll \
    && make \
    && make install \
    && rm -rf /tmp/hll )

# Install PL languages
RUN apt-get install -y postgresql-plpython3-17 postgresql-plperl-17



# Install pgrouting extension
RUN apt-get install -y postgresql-17-pgrouting || \
    ( apt-get install -y cmake libboost-graph-dev \
    && git clone https://github.com/pgRouting/pgrouting.git /tmp/pgrouting \
    && cd /tmp/pgrouting \
    && mkdir build && cd build \
    && cmake .. \
    && make \
    && make install \
    && rm -rf /tmp/pgrouting )



# Install ip4r extension
RUN apt-get install -y postgresql-17-ip4r || \
    ( git clone https://github.com/RhodiumToad/ip4r.git /tmp/ip4r \
    && cd /tmp/ip4r \
    && make \
    && make install \
    && rm -rf /tmp/ip4r )

# Install postgresql-hll extension
RUN if ! dpkg -l | grep -q postgresql-17-hll; then \
      git clone https://github.com/citusdata/postgresql-hll.git /tmp/postgresql-hll \
      && cd /tmp/postgresql-hll \
      && make \
      && make install \
      && rm -rf /tmp/postgresql-hll; \
    fi



# Install GEOS support for PostGIS
RUN apt-get install -y libgeos-dev

# Clean up: Remove unnecessary packages and files
RUN apt-get remove -y make gcc git-all postgresql-server-dev-17 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create initialization script for shared_preload_libraries
RUN echo "# Remember to add the following to postgresql.conf:" > /docker-entrypoint-initdb.d/extension-config.txt \
    && echo "# shared_preload_libraries = 'pg_stat_statements,pg_cron,pgaudit'" >> /docker-entrypoint-initdb.d/extension-config.txt

