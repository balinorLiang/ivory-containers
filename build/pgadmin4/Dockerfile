ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
ARG BASE_IMAGE_NAME
#***************************************************
# Create env builder
#***************************************************
FROM ${BASE_IMAGE_NAME}:${BASEOS}-${PG_FULL}-${BASEVER} as pg15-builder

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG BASEOS
ARG PACKAGER
ARG PG_MAJOR
ARG PGADMIN_VER
ARG IVY_MAJOR

RUN ${PACKAGER} -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
RUN ${PACKAGER} -y install --nodocs \
                --setopt=skip_missing_names_on_install=False \
                soci-postgresql-devel \
                postgresql${PG_MAJOR//.}-server

#***************************************************
# Create pgadmin builder
#***************************************************
FROM ${BASE_IMAGE_NAME}:${BASEOS}-${PG_FULL}-${BASEVER}  as pgadmin-builder

ARG BASEOS
ARG PACKAGER
ARG PG_MAJOR
ARG PGADMIN_VER
ARG IVY_MAJOR
RUN mkdir -p /var/lib/pgadmin

RUN rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-redhat-repo-2-1.noarch.rpm 
RUN ${PACKAGER} -y install --nodocs \
                --setopt=skip_missing_names_on_install=False \
                --setopt=obsoletes=0 pgadmin4-web-7.4 

RUN cp -rf /usr/pgadmin4/venv/lib/python3.9/site-packages/* /usr/lib/python3.9/site-packages/ \
    && cp -rp /usr/pgadmin4/web /usr/lib/python3.9/site-packages/pgadmin4-web

#***************************************************
# Create final builder
#***************************************************
FROM redhat/ubi8:latest

ARG BASEOS
ARG PACKAGER
ARG PG_MAJOR
ARG PGADMIN_VER
ARG IVY_MAJOR

COPY --from=pg15-builder /usr/pgsql-15/bin/pg_dump /usr/bin/pg_dump
COPY --from=pg15-builder /usr/pgsql-15/bin/pg_dumpall /usr/bin/pg_dumpall
COPY --from=pg15-builder /usr/pgsql-15/bin/pg_restore /usr/bin/pg_restore
COPY --from=pg15-builder /usr/pgsql-15/bin/psql /usr/bin/psql
COPY --from=pg15-builder /usr/lib64/libpq.so.5.13 /usr/lib64

COPY --from=pgadmin-builder /usr/lib/python3.9 /usr/lib/python3.9
COPY --from=pgadmin-builder /usr/bin/python3.9 /usr/bin/python3.9
COPY --from=pgadmin-builder /usr/bin/pip3.9 /usr/bin/pip3.9
COPY --from=pgadmin-builder /usr/lib64/libpython3.9.so.1.0 /usr/lib64/
COPY --from=pgadmin-builder /usr/lib64/python3.9 /usr/lib64/python3.9
COPY --from=pgadmin-builder /var/lib/pgadmin/ /var/lib/pgadmin/
COPY --from=pgadmin-builder /etc/httpd/modules/mod_wsgi_python3.so /

LABEL name="pgadmin4" \
        summary="pgAdmin4 GUI utility" \
        description="Provides GUI for the pgAdmin 4 utility." \
        io.k8s.description="pgadmin4 container" \
        io.k8s.display-name="pgAdmin 4" \
        io.openshift.tags="postgresql,postgres,monitoring,pgadmin4,pgadmin,database,crunchy"

RUN ln -s libpq.so.5.13 /usr/lib64/libpq.so.5 && \
    ln -s libpq.so.5.13 /usr/lib64/libpq.so

RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /var/log/pgadmin \
        /certs /run/httpd

RUN sed -e "s|^mirrorlist=|#mirrorlist=|g" \
         -e "s|^#baseurl=http://mirror.centos.org/\$contentdir/\$releasever|baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos-vault/8.4.2105|g" \
         -i.bak \
         /etc/yum.repos.d/ubi.repo \
	&& yum makecache

RUN ${PACKAGER} -y install --nodocs \
                mod_ssl \
                python3-gssapi \
                openssl \
                sqlite \
                nss_wrapper \
                python3-mod_wsgi \
		gettext \
        && ${PACKAGER} -y clean all 

RUN mv /mod_wsgi_python3.so /etc/httpd/modules/

RUN chown -R 2:0 /usr/lib/python3.9/site-packages/pgadmin4-web \
                /var/lib/pgadmin /var/log/pgadmin /certs /etc/httpd /run/httpd \
                /var/log/httpd \
        && chmod -R g=u /usr/lib/python3.9/site-packages/pgadmin4-web \
                /var/lib/pgadmin /var/log/pgadmin /certs /etc/httpd /run/httpd /var/log/httpd \
        && ln -sf /var/lib/pgadmin/config_local.py /usr/lib/python3.9/site-packages/pgadmin4-web/config_local.py \
        && ln -sf /var/lib/pgadmin/pgadmin.conf /etc/httpd/conf.d/pgadmin.conf \
        && ln -sf /usr/bin/python3.9 /usr/bin/python3 \
        && ln -sf /usr/bin/pip3.9 /usr/bin/pip3 \
        && ln -sf /usr/bin/python3.9 /usr/bin/python

# Preserving PGVERSION out of paranoia
ENV PGROOT="/usr/pgsql-${PG_MAJOR}" PGVERSION="${PG_MAJOR}"

ADD bin/pgadmin4/ /opt/crunchy/bin
ADD bin/common /opt/crunchy/bin
ADD conf/pgadmin4/ /opt/crunchy/conf

RUN cp /opt/crunchy/conf/httpd.conf /etc/httpd/conf/httpd.conf \
        && rm /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/ssl.conf

EXPOSE 5050

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/var/lib/pgadmin", "/certs", "/run/httpd"]

# Defines a unique directory name that will be utilized by the nss_wrapper in the UID script
ENV NSS_WRAPPER_SUBDIR="pgadmin"

ENTRYPOINT ["opt/crunchy/bin/uid_daemon.sh"]

USER 2

CMD ["/opt/crunchy/bin/start-pgadmin4.sh"]

