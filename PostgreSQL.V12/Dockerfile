######
# PostgreSQL Dockerfile on redhat 7 / CentOS 7
#

# Build:
# docker build -t zokeber/postgresql:latest .
#
# Create:
# docker create -it -p 5432:5432 --name postgresqlDB DB/postgresql
#
# Start:
# docker start postgresqlDB
#
# Connect with postgresql client
# docker exec -it postgresqlDB psql
#
# Connect bash
# docker exec -it postgresqlDB bash


# Pull base image
# FROM https://hub.docker.com/_/postgres
FROM postgres:12.2

# Postgresql version
ENV PG_VERSION 12.2
ENV PGVERSION 12

COPY rhel-7-server-rpms.repo /etc/yum.repos.d/
COPY amd-external-pci-pmx-10.3.repo /etc/yum.repos.d/
ADD zoneinfo /usr/share/zoneinfo

# Set the environment variables
ENV HOME /var/lib/pgsql/12
ENV PGDATA /var/lib/pgsql/12/data

RUN microdnf --enablerepo=rhel-7-server-rpms --enablerepo=amd-external-pci-pmx-10.3 install -y systemd && \
    microdnf --enablerepo=rhel-7-server-rpms --enablerepo=amd-external-pci-pmx-10.3 install -y postgresql12-server-12.2 && \
    microdnf --enablerepo=rhel-7-server-rpms --enablerepo=amd-external-pci-pmx-10.3 install -y postgresql12-contrib-12.2 && \
    microdnf clean all && \
    rm -f /etc/yum.repos.d/amd-external-pci-pmx-10.3.repo

WORKDIR /usr/pgsql-12
RUN cp /usr/pgsql-12/bin/postgresql-12-setup /usr/bin/
RUN cp /usr/pgsql-12/bin/pg_ctl /usr/bin/

# Working directory
WORKDIR /var/lib/pgsql/12

# Run initdb
RUN rm -fr /var/lib/pgsql/12/data
RUN chmod +x /usr/bin/postgresql-12-setup
ENV PGSETUP_INITDB_OPTIONS "--pgdata=/var/lib/pgsql/12/data --encoding=UTF8"
ENV PGDATA /var/lib/pgsql/12/data
#RUN /usr/bin/postgresql-12-setup initdb 
USER postgres
RUN /usr/pgsql-12/bin/initdb -D /var/lib/pgsql/12/data -A trust -U postgres --pgdata=/var/lib/pgsql/12/data --encoding=UTF8
##RUN chmod +x /usr/pgsql-12/bin/postgresql-12-setup
##RUN /usr/pgsql-12/bin/postgresql-12-setup initdb 


# Copy config file
COPY data/postgresql.sh /usr/local/bin/postgresql.sh
COPY data/postgresql.conf /var/lib/pgsql/12/data/postgresql.conf 
COPY data/pg_hba.conf /var/lib/pgsql/12/data/pg_hba.conf 

USER root
# Change own user
RUN chown -R postgres:postgres /var/lib/pgsql/12/data/* && \
    usermod -G wheel postgres # && \
#    chmod +x /usr/local/bin/postgresql

# Set volume
VOLUME ["/var/lib/pgsql/12"]

# Set username
USER root
#USER postgres

# Run PostgreSQL Server
RUN chmod +x /usr/local/bin/postgresql.sh
RUN chown postgres:postgres /usr/local/bin/postgresql.sh
USER postgres
CMD ["/bin/bash", "/usr/local/bin/postgresql.sh"]
ENTRYPOINT ["/usr/local/bin/postgresql.sh"]
#CMD ["sudo service postgresql start"]
##RUN rm -fr /usr/local/pgsql/data
##ADD data /usr/local/pgsql/data
##RUN /usr/pgsql-12/bin/pg_ctl start # -D /usr/local/pgsql/data

# Expose ports
EXPOSE 5432

