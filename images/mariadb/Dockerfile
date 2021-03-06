ARG IMAGE_REPO
FROM ${IMAGE_REPO:-lagoon}/commons as commons
FROM alpine

MAINTAINER amazee.io

# Copying commons files
COPY --from=commons /lagoon /lagoon
COPY --from=commons /bin/fix-permissions /bin/ep /bin/docker-sleep /bin/
COPY --from=commons /home /home

# When Bash is invoked via `sh` it behaves like the old Bourne Shell and sources a file that is given in `ENV`
# When Bash is invoked as non-interactive (like `bash -c command`) it sources a file that is given in `BASH_ENV`
ENV TMPDIR=/tmp TMP=/tmp HOME=/home ENV=/home/.bashrc BASH_ENV=/home/.bashrc

MAINTAINER amazee.io
ENV BACKUPS_DIR="/var/lib/mysql/backup"

ENV MARIADB_DATABASE=lagoon \
    MARIADB_USER=lagoon \
    MARIADB_PASSWORD=lagoon

RUN apk add --no-cache bash mariadb-client mariadb mariadb-common pwgen && rm -rf /var/cache/apk/*

RUN for i in /run/mysqld /var/lib/mysql /etc/mysql/conf.d /docker-entrypoint-initdb.d/ "${BACKUPS_DIR}"; \
do mkdir -p $i; chown mysql $i;  /bin/fix-permissions $i; done

COPY docker-entrypoint.sh /lagoon/entrypoints/90-mariadb-entrypoint
COPY mysql-backup.sh /lagoon/

COPY my.cnf /etc/mysql/my.cnf

RUN ln -s /var/lib/mysql/.my.cnf /home/.my.cnf

### we cannot start mysql as root, we add the user mysql to the group root and change the user of the Docker Image to this user
USER mysql

WORKDIR /var/lib/mysql
VOLUME /var/lib/mysql
EXPOSE 3306

ENTRYPOINT ["/lagoon/entrypoints.sh"]
CMD ["mysqld"]
