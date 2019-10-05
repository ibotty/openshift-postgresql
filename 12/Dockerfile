FROM centos:7

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

ENV POSTGRESQL_VERSION=12 \
    HOME=/var/lib/pgsql \
    PGUSER=postgres \
    POSTGRES_RELEASE_REPO=https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

LABEL io.k8s.description="PostgreSQL is an advanced Object-Relational database management system" \
      io.k8s.display-name="PostgreSQL 12" \
      io.openshift.expose-services="5432:postgresql" \
      io.openshift.tags="database,postgresql,postgresql12"

EXPOSE 5432

# This image must forever use UID 26 for postgres user so our volumes are
# safe in the future. This should *never* change, the last test is there
# to make sure of that.
RUN yum install -y $POSTGRES_RELEASE_REPO epel-release centos-release-scl-rh && \
    yum -y --setopt=tsflags=nodocs install gettext bind-utils nss_wrapper \
        postgresql12-server postgresql12-contrib postgresql12-devel pgxnclient \
        pg_activity rsync openssh llvm-toolset-7 llvm5.0 \
        make gcc python-setuptools && \
    export PATH=/usr/pgsql-${POSTGRESQL_VERSION}/bin:/usr/bin:/bin:/usr/sbin:/sbin && \
    pgxn install url_encode && \
    pgxn install first_last_agg && \
    echo yum remove -y make gcc python-setuptools *-devel pgxnclient llvm-* cmake devtools-* kernel-headers && \
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
