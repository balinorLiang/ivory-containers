ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
ARG BASE_IMAGE_NAME
FROM redhat/ubi8:latest

ARG PG_MAJOR

RUN yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

RUN yum install -y gettext \
        && yum -y clean all

RUN yum -y install --nodocs \
                --setopt=skip_missing_names_on_install=False \
                postgresql${PG_MAJOR//.}-server \
        && yum -y clean all

RUN mkdir -p /opt/cpm/bin /opt/cpm/conf

COPY conf/pgexporter/common /opt/cpm/conf
COPY conf/pgexporter/linux /opt/cpm/conf
COPY bin/pgexporter /opt/cpm/bin

RUN chgrp -R 0 /opt/cpm/bin /opt/cpm/conf && \
        chmod -R g=u /opt/cpm/bin/ opt/cpm/conf

RUN chmod +x /opt/cpm/bin/start.sh

EXPOSE 9187

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/conf"]

USER 2

CMD ["/opt/cpm/bin/start.sh"]
