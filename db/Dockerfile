FROM postgres:15.6

RUN apt-get update && apt-get install -y gettext-base

COPY init.sql /docker-entrypoint-initdb.d/init.sql.raw

RUN mkdir -p /etc/postgresql/ \
    && mkdir -p /oracle/pg_data/archive/ \
    && chown postgres:postgres /oracle/pg_data/archive/

ENTRYPOINT [ "bash", "-c", "\
mkdir -p /var/log/postgresql/ \n\
chown postgres:postgres /var/log/postgresql/ \n\
chmod 777 /var/log/postgresql/ \n\
envsubst < /docker-entrypoint-initdb.d/init.sql.raw > /docker-entrypoint-initdb.d/init.sql \n\
touch /var/log/postgresql/postgresql.log \n\
chmod 777 /var/log/postgresql/postgresql.log \n\
cat <<EOF > /etc/postgresql/postgresql.conf \n\
listen_addresses = '*' \n\
port = ${DB_PORT} \n\
log_destination = 'stderr' \n\
logging_collector = on \n\
log_directory = '/var/log/postgresql/' \n\
log_filename = 'postgresql.log' \n\
archive_mode = on \n\
archive_command = 'cp %p /oracle/pg_data/archive/%f' \n\
max_wal_senders = 10 \n\
wal_level = replica \n\
wal_log_hints = on \n\
log_replication_commands = on \n\
hot_standby = on \n\
EOF\n\
cat <<EOF > /etc/postgresql/pg_hba.conf \n\
local all all trust \n\
host all all 0.0.0.0/0 md5 \n\
host replication ${DB_REPL_USER} 192.168.1.12/16 trust \n\
EOF\n\
exec docker-entrypoint.sh $@" ]

CMD ["postgres", "-c", "config_file=/etc/postgresql/postgresql.conf", "-c", "hba_file=/etc/postgresql/pg_hba.conf"]
