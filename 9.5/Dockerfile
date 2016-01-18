FROM centos:centos7

# PostgreSQL image for OpenShift.
# Volumes:
#  * /var/lib/psql/data   - Database cluster for PostgreSQL
# Environment:
#  * $POSTGRESQL_USER     - Database user name
#  * $POSTGRESQL_PASSWORD - User's password
#  * $POSTGRESQL_DATABASE - Name of the database to create
#  * $POSTGRESQL_ADMIN_PASSWORD (Optional) - Password for the 'postgres'
#                           PostgreSQL administrative account

MAINTAINER SoftwareCollections.org <sclorg@redhat.com>

ENV POSTGRESQL_VERSION=9.5 \
    HOME=/var/lib/pgsql \
    PGUSER=postgres \
    POSTGRES_RELEASE_REPO=http://yum.postgresql.org/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm

LABEL io.k8s.description="PostgreSQL is an advanced Object-Relational database management system" \
      io.k8s.display-name="PostgreSQL 9.5" \
      io.openshift.expose-services="5432:postgresql" \
      io.openshift.tags="database,postgresql,postgresql95"

EXPOSE 5432

# This image must forever use UID 26 for postgres user so our volumes are
# safe in the future. This should *never* change, the last test is there
# to make sure of that.
RUN rpm -Uvh $POSTGRES_RELEASE_REPO && \
    yum -y install epel-release && \
    rpmkeys --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 && \
    yum -y --setopt=tsflags=nodocs install gettext bind-utils nss_wrapper \
        postgresql95-server postgresql95-devel pgxnclient pg_activity \
        make gcc && \
    yum clean all && \
    localedef -f UTF-8 -i en_US en_US.UTF-8 && \
    mkdir -p /var/lib/pgsql/data /etc/credentials && \
    test "$(id postgres)" = "uid=26(postgres) gid=26(postgres) groups=26(postgres)"

# Loosen permission bits to avoid problems running container with arbitrary UID
ADD root /
RUN /usr/libexec/fix-permissions /var/lib/pgsql && \
    /usr/libexec/fix-permissions /var/run/postgresql

# Get prefix path and path to scripts rather than hard-code them in scripts
ENV CONTAINER_SCRIPTS_PATH=/usr/share/container-scripts/postgresql

# When bash is started non-interactively, to run a shell script, for example it
# looks for this variable and source the content of this file. This will enable
# the SCL for all scripts without need to do 'scl enable'.
ENV BASH_ENV=${CONTAINER_SCRIPTS_PATH}/scl_enable \
    ENV=${CONTAINER_SCRIPTS_PATH}/scl_enable \
    PROMPT_COMMAND=". ${CONTAINER_SCRIPTS_PATH}/scl_enable"

VOLUME ["/var/lib/pgsql/data"]

USER 26

ENTRYPOINT ["container-entrypoint"]
CMD ["run-postgresql"]