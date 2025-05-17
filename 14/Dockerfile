# Start with the official PostgreSQL 14 image
FROM postgres:14

# Update the apt package list and install dependencies needed for building extensions,
RUN apt-get update \
    && apt-get install -y \
       make gcc git-all postgresql-server-dev-14 postgresql-14-pglogical postgresql-14-cron \
       libkrb5-dev libssl-dev zlib1g-dev libreadline-dev libxml2-dev libxslt1-dev libpq-dev wget unzip \
       build-essential curl libedit-dev libpam0g-dev libpython3-dev uuid-dev lsb-release \
    && rm -rf /var/lib/apt/lists/*

# Install the pglogical extension (for logical replication in PostgreSQL 14)
RUN apt-get install -y postgresql-14-pglogical

# Install the pg_cron extension (for cron jobs within PostgreSQL), needs shared_preload_libraries config
RUN git clone --branch v1.6.3 https://github.com/citusdata/pg_cron.git /tmp/pg_cron \
    && cd /tmp/pg_cron \
    && make \
    && make install \
    && rm -rf /tmp/pg_cron
    
# Install aiven-extras extension
RUN git clone https://github.com/aiven/aiven-extras.git /tmp/aiven-extras \
    && cd /tmp/aiven-extras \
    && make \
    && make install \
    && rm -rf /tmp/aiven-extras


# Add the PostgreSQL Global Development Group (PGDG) APT repository and import its signing key.
# lsb-release is now installed so $(lsb_release -cs) works correctly.
RUN echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
    && apt-get update

# Install PostGIS extension (for geographic object support)
RUN apt-get install -y postgresql-14-postgis-3

# Install pg_hint_plan extension (for query optimization hints in PostgreSQL)
RUN git clone https://github.com/ossc-db/pg_hint_plan.git /tmp/pg_hint_plan \
    && cd /tmp/pg_hint_plan \
    && git checkout REL14_1_4_3 \
    && make \
    && make install \
    && rm -rf /tmp/pg_hint_plan

# Install pgrouting extension (for routing functionality in PostgreSQL)
RUN apt-get install -y postgresql-14-pgrouting

# Install pg_partman extension (for partition management in PostgreSQL)
RUN git clone https://github.com/pgpartman/pg_partman.git /tmp/pg_partman \
    && cd /tmp/pg_partman \
    && git checkout v4.7.3 \
    && make \
    && make install \
    && rm -rf /tmp/pg_partman

# Install wal2json extension (for logical replication in JSON format)
RUN git clone https://github.com/eulerto/wal2json.git /tmp/wal2json \
    && cd /tmp/wal2json \
    && make \
    && make install \
    && rm -rf /tmp/wal2json

# Install other PostgreSQL 14 extensions available as packages (excluding PLV8)
RUN apt-get install -y \
    postgresql-14-ip4r postgresql-14-repack \
    postgresql-14-decoderbufs postgresql-14-orafce postgresql-14-hypopg \
    postgresql-14-hll postgresql-14-pg-qualstats postgresql-14-pg-stat-kcache

# Install PL/Python extension
RUN apt-get install -y postgresql-plpython3-14

# Install additional extensions from source

# Install pg_freespacemap
RUN cd /usr/share/postgresql/14/extension/ \
    && if [ ! -f pg_freespacemap--1.2.sql ]; then \
         cd /tmp \
         && curl -O https://ftp.postgresql.org/pub/source/v14.14/postgresql-14.14.tar.gz \
         && tar -xzf postgresql-14.14.tar.gz \
         && cd postgresql-14.14 \
         && ./configure \
         && cd contrib/pg_freespacemap \
         && make \
         && make install \
         && cd /tmp \
         && rm -rf postgresql-14.14 postgresql-14.14.tar.gz; \
       fi


# Install pg_repack
RUN if ! dpkg -l | grep -q postgresql-14-repack; then \
      git clone https://github.com/reorg/pg_repack.git /tmp/pg_repack \
      && cd /tmp/pg_repack \
      && git checkout ver_1.4.8 \
      && make \
      && make install \
      && rm -rf /tmp/pg_repack; \
    fi

# Install pgAudit
RUN git clone https://github.com/pgaudit/pgaudit.git /tmp/pgaudit \
    && cd /tmp/pgaudit \
    && git checkout REL_14_STABLE \
    && make USE_PGXS=1 PG_CONFIG=/usr/bin/pg_config \
    && make install USE_PGXS=1 PG_CONFIG=/usr/bin/pg_config \
    && rm -rf /tmp/pgaudit


# Install pgVector for vector similarity search
RUN git clone --branch v0.4.0 https://github.com/pgvector/pgvector.git /tmp/pgvector \
    && cd /tmp/pgvector \
    && make \
    && make install \
    && rm -rf /tmp/pgvector

# Install pg_prewarm
RUN cd /usr/share/postgresql/14/extension/ \
    && if [ ! -f pg_prewarm--1.2.sql ]; then \
         cd /tmp \
         && curl -O https://ftp.postgresql.org/pub/source/v14.14/postgresql-14.14.tar.gz \
         && tar -xzf postgresql-14.14.tar.gz \
         && cd postgresql-14.14 \
         && ./configure \
         && cd contrib/pg_prewarm \
         && make \
         && make install \
         && cd /tmp \
         && rm -rf postgresql-14.14 postgresql-14.14.tar.gz; \
       fi

# Install plperl
RUN apt-get install -y postgresql-plperl-14

# Install tablefunc
RUN cd /usr/share/postgresql/14/extension/ \
    && if [ ! -f tablefunc--1.0.sql ]; then \
       cd /tmp \
       && curl -O https://ftp.postgresql.org/pub/source/v14.14/postgresql-14.14.tar.gz \
       && tar -xzf postgresql-14.14.tar.gz \
       && cd postgresql-14.14/contrib/tablefunc \
       && make \
       && make install \
       && cd /tmp \
       && rm -rf postgresql-14.14 postgresql-14.14.tar.gz; \
    fi

# Install btree_gin and btree_gist
RUN cd /usr/share/postgresql/14/extension/ \
    && if [ ! -f btree_gin--1.3.sql ]; then \
         cd /tmp \
         && curl -O https://ftp.postgresql.org/pub/source/v14.14/postgresql-14.14.tar.gz \
         && tar -xzf postgresql-14.14.tar.gz \
         && cd postgresql-14.14 \
         && ./configure \
         && cd contrib/btree_gin \
         && make \
         && make install \
         && cd ../btree_gist \
         && make \
         && make install \
         && cd /tmp \
         && rm -rf postgresql-14.14 postgresql-14.14.tar.gz; \
       fi

# Clean up: Remove unnecessary packages and files
RUN apt-get remove -y make gcc git-all postgresql-server-dev-14 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Add note for pg_cron and pgAudit configuration: user needs to add it to postgresql.conf
RUN echo "# Remember to add the following to postgresql.conf:" > /docker-entrypoint-initdb.d/extension-config.txt \
    && echo "# shared_preload_libraries = 'pg_cron,pgaudit'" >> /docker-entrypoint-initdb.d/extension-config.txt

