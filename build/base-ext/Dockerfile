ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
FROM ${PREFIX}/highgo-base:${BASEOS}-${PG_FULL}-${BASEVER}

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG BASEOS
ARG PACKAGER

LABEL name="highgo-base-ext" \
	summary="Includes base extensions required to load in additional PostgreSQL extensions." \
	description="Includes base extensions required to load in additional PostgreSQL extensions."

RUN if [ "$BASEOS" = "centos8" ] ; then \
	${PACKAGER} -y install --nodocs \
		--setopt=skip_missing_names_on_install=False \
		--enablerepo="powertools" \
		perl \
	&& ${PACKAGER} -y clean all ; \
fi

#RUN if [ "$BASEOS" = "ubi8" ] ; then \
#	${PACKAGER} -y --enablerepo="epel" --enablerepo="ubi-8-codeready-builder-rpms" --nodocs install libaec libdap armadillo \
#	&& ${PACKAGER} -y install --nodocs \
#		--enablerepo="epel" \
#		hdf5 \
#		openldap \
#		perl \
#	&& ${PACKAGER} -y clean all --enablerepo="epel" --enablerepo="ubi-8-codeready-builder-rpms" ; \
#fi
