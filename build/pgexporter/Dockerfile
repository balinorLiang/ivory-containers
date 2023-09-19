ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
FROM highgo-base:${BASEOS}-${PG_FULL}-${BASEVER}

ARG BASEOS
ARG PACKAGER
ARG PG_MAJOR

RUN rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-redhat-repo-2-1.noarch.rpm \
        && yum -y update epel-release \
        && yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm \
        && yum -y makecache

RUN yum install -y wget

RUN yum -y install --nodocs \
                --setopt=skip_missing_names_on_install=False \
                mod_ssl \
                python3-gssapi \
                openssl \
                sqlite \
                make \
                epel-release \
                soci-postgresql-devel \
                krb5-devel \
                python3-devel \
                sqlite-devel \
                gcc \
                postgresql${PG_MAJOR//.}-server \
        && yum -y clean all

RUN mkdir -p /opt/cpm/bin /opt/cpm/conf

RUN wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.13.2/postgres_exporter-0.13.2.linux-amd64.tar.gz \
-O /opt/cpm/bin/postgres_exporter.tar.gz

COPY conf/pgexporter/common /opt/cpm/conf
COPY conf/pgexporter/linux /opt/cpm/conf
COPY bin/pgexporter /opt/cpm/bin

RUN chgrp -R 0 /opt/cpm/bin /opt/cpm/conf && \
        chmod -R g=u /opt/cpm/bin/ opt/cpm/conf

EXPOSE 9187

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/conf"]

USER 2

CMD ["/opt/cpm/bin/start.sh"]
