FROM postgres:17

RUN apt-get update \
    && apt-get install -y make gcc git-all postgresql-server-dev-17 postgresql-17-pglogical

RUN git clone https://github.com/aiven/aiven-extras.git /tmp/aiven-extras \
    && cd /tmp/aiven-extras \
    && make \
    && make install \
    && rm -rf /tmp/aiven-extras

RUN git clone --branch v1.6.3 https://github.com/citusdata/pg_cron.git /tmp/pg_cron \
    && cd /tmp/pg_cron \
    && make \
    && make install \
    && rm -rf /tmp/pg_cron

RUN git clone https://github.com/aiven/aiven-extras.git /tmp/aiven-extras \
    && cd /tmp/aiven-extras \
    && make \
    && make install \
    && rm -rf /tmp/aiven-extras

RUN git clone --depth 1 --branch <tag_name> <repo_url>

RUN apt-get remove -y make gcc git-all postgresql-server-dev-17 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

