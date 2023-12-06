ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
ARG BASE_IMAGE_NAME
ARG IVYO_VER
FROM ${BASE_IMAGE_NAME}:${BASEOS}-${PG_FULL}-${IVYO_VER}-${BASEVER}
#FROM highgo-base:centos8-15.2-0.0.1-0

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG PG_FULL
ARG BASEOS
ARG DFSET
ARG PACKAGER

# ===== Early lines ordered for leveraging cache, reorder carefully =====
ARG PG_MAJOR
ARG IVY_MAJOR
# Needed due to lack of environment substitution trick on ADD
ARG PG_LBL

RUN ${PACKAGER} -y update epel-release \
	&& ${PACKAGER} -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm \
#	&& ${PACKAGER} -y clean all
	&& ${PACKAGER} -y makecache

# Preserving PGVERSION out of paranoia
ENV PGROOT="/usr/pgsql-${PG_MAJOR}" PGVERSION="${PG_MAJOR}" IVYROOT="/usr/local/ivorysql/ivorysql-${IVY_MAJOR}" IVYVERSION="${IVY_MAJOR}" 

ARG PATRONI_VER
ARG BACKREST_VER

# Separate yum run matching postgres-ha image, leverages cached layer
RUN if [ "$DFSET" = "centos" ] ; then \
        ${PACKAGER} -y install \
        	--setopt=skip_missing_names_on_install=False \
        	openssh-clients \
        	openssh-server \
#        	pgaudit17_15 \
#	       	set_user_15 \
#        	pg_partman_${PG_MAJOR//.} \
#        	pg_cron_${PG_MAJOR//.} \
#		pgbackrest \
		pgbackrest-2.45 \
#        	crunchy-backrest-${BACKREST_VER} \
#        	postgresql${PG_MAJOR//.}-contrib \
#        	postgresql${PG_MAJOR//.}-server \
#        	postgresql${PG_MAJOR//.}-plpython* \
                ivorysql2-devel \
                ivorysql2-contrib \
                ivorysql2-server \
                ivorysql2-plpython* \
#        	pgnodemx${PG_MAJOR//.} \
#        	$( printf '11\n'${PG_MAJOR} | sort -VC && echo postgresql${PG_MAJOR}-llvmjit ) \
        	psmisc \
        	python3-pip \
        	python3-psutil \
        	python3-psycopg2 \
        	rsync \
 #       	timescaledb_${PG_MAJOR//.} \
        	wal2json_${PG_MAJOR//.} \
        	file \
        	gettext \
        	hostname \
        	procps-ng \
        	pgaudit_analyze \
        	unzip \
        	bzip2 \
        	lz4 \
		krb5-workstation \
#		orafce_${PG_MAJOR//.} \
#		passwordpolicy \
        && ${PACKAGER} -y clean all ; \
else \
        ${PACKAGER} -y install \
		--enablerepo="epel" \
		openssh-clients \
		openssh-server \
#		pgaudit${PG_MAJOR//.} \
#		pgaudit${PG_MAJOR//.}_set_user \
#		pg_partman_${PG_MAJOR//.} \
#		pg_cron_${PG_MAJOR//.} \
		pgbackrest-2.45 \
#		postgresql${PG_MAJOR//.}-contrib \
#		postgresql${PG_MAJOR//.}-server \
#		postgresql${PG_MAJOR//.}-plpython* \
#                ivorysql2-devel \
                ivorysql2-contrib \
                ivorysql2-server \
                ivorysql2-plpython3 \
		ivorysql2-pltcl \
		ivorysql2-plperl \
#		pgnodemx${PG_MAJOR//.} \
#		$( printf '11\n'${PG_MAJOR} | sort -VC && echo postgresql${PG_MAJOR}-llvmjit ) \
		psmisc \
		python3-pip \
		python3-psutil \
		python3-psycopg2 \
		rsync \
#		timescaledb_${PG_MAJOR//.} \
#		wal2json_${PG_MAJOR//.} \
		file \
		unzip \
		tar \
		bzip2 \
		lz4 \
		krb5-workstation \
#		orafce_${PG_MAJOR//.} \
#		passwordpolicy \
#	&& ${PACKAGER} -y install --nodocs \
#		--setopt=tsflags='' \
#		--enablerepo="epel" \
#		pgaudit_analyze \
#	&& ${PACKAGER} -y clean all --enablerepo="epel" ; \
	&& ${PACKAGER} -y clean all ; \ 
fi

RUN cat /dev/null > /etc/pgbackrest.conf

RUN  ${PACKAGER} -y install \
	redhat-rpm-config \
	openssl-devel \
	make \
	&& ${PACKAGER} -y clean all
# insatll pgaudit
ADD rpms/pgaudit-1.7.0.tar.gz /
RUN cd pgaudit-1.7.0 \
	&& make install USE_PGXS=1 PG_CONFIG=$IVYROOT/bin/pg_config \
	&& rm -rf ../pgaudit-1.7.0
# insatll pg_cron
#RUN yum install -y pg_cron_15 --installroot=$IVYROOT  --releasever=8 --skip-broken
ADD rpms/pg_cron-main.tar.gz /
RUN cd pg_cron-main \
#	&& sed -i 's/-Wno-compound-token-split-by-macro //g' $IVYROOT/lib/pgxs/src/Makefile.global \
        && make USE_PGXS=1 PG_CONFIG=$IVYROOT/bin/pg_config \
        && make install USE_PGXS=1 PG_CONFIG=$IVYROOT/bin/pg_config \
	&& rm -rf ../pg_cron-main/

# install patroni for Kube
RUN pip3 install --upgrade python-dateutil \
	&& pip3 install patroni[kubernetes]=="${PATRONI_VER}" -i https://pypi.douban.com/simple/

ENV PATH="${IVYROOT}/bin:${PATH}"

# install timescaledb
ADD rpms/timescaledb-2.11.1.tar.gz /
RUN cd timescaledb-2.11.1 \
        && ./bootstrap \
        && cd build \
        && make PG_CONFIG=$IVYROOT/bin/pg_config \
        && make install PG_CONFIG=$IVYROOT/bin/pg_config

LABEL name="ivory" \
	summary="Highgo IvorySQL" \
	description="Ivory ready for production: high availability, disaster recovery, monitoring, security, and performance. Works with IVYO, the open source Ivory Operator from Highgo." \
	io.k8s.description="Highgo IvorySQL is ready for production: the trusted open source distribution of PostgreSQL. Works with IVYO, the open source Ivory Operator from Highgo." \
	io.k8s.display-name="Highgo IvorySQL" \
	io.openshift.tags="ivorysql,postgresql,postgres,postgis,sql,nosql,database,ha,highgo" \
	postgresql.version.major="${PG_MAJOR}" \
	postgresql.version="${PG_FULL}"

# set up crunchy directory
RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /pgdata /pgwal /pgconf /backrestrepo

RUN chown -R ivorysql:ivorysql /opt/crunchy /var/lib/pgsql \
		/pgdata /pgwal /pgconf /backrestrepo &&  \
	chmod -R g=u /opt/crunchy /var/lib/pgsql \
		/pgdata /pgwal /pgconf /backrestrepo

# install wal2json
ADD rpms/wal2json_2_5.tar.gz /
RUN cd wal2json-wal2json_2_5 \
    && export PATH=$IVYROOT/bin:$PATH \
    && make \
    && make install \
    && rm -rf ../wal2json-wal2json_2_5

# open up the postgres port
EXPOSE 5432 5866

ADD bin/postgres_common /opt/crunchy/bin
ADD bin/common /opt/crunchy/bin
ADD conf/postgres_common /opt/crunchy/conf
ADD tools/pgmonitor/postgres_exporter/common /opt/crunchy/bin/modules/pgexporter
ADD tools/pgmonitor/postgres_exporter/linux /opt/crunchy/bin/modules/pgexporter

# install pgnodemx
ADD rpms/pgnodemx-1.5.tar.gz /
RUN cd pgnodemx-1.5 \
    && make USE_PGXS=1 \
    && make USE_PGXS=1 install

RUN mkdir /.ssh && chown 998:0 /.ssh && chmod g+rwx /.ssh && rm -f /run/nologin

# remove the default spool directory so that pgBackRest does not attempt to look there when
# performing a restore (pgBackRest will not have permissions to access to this dir in all envs)
RUN rm -rf /var/spool/pgbackrest

# add volumes to allow override of pg_hba.conf and postgresql.conf
# add volumes to allow storage of postgres WAL segment files
# add volumes to locate WAL files to recover with
# add volumes for pgbackrest to write to
# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/pgdata", "/pgwal", "/pgconf", "/backrestrepo", "/sshd"]

# Defines a unique directory name that will be utilized by the nss_wrapper in the UID script
ENV NSS_WRAPPER_SUBDIR="ivorysql"

ENTRYPOINT ["/opt/crunchy/bin/uid_postgres.sh"]

USER 998
#USER 0

CMD ["/opt/crunchy/bin/start.sh"]
