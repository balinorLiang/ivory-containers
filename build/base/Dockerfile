
ARG BASE_IMAGE_OS
ARG DOCKERBASEREGISTRY
FROM ${DOCKERBASEREGISTRY}${BASE_IMAGE_OS}

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG BASEOS
ARG RELVER
ARG PACKAGER

MAINTAINER Highgo <support@highgo.com>

LABEL vendor="Highgo" \
	url="https://www.highgo.com/" \
	release="${RELVER}" \
	org.opencontainers.image.vendor="Highgo" \
	os.version="7.9"

COPY redhat/atomic/help.1 /help.1
COPY redhat/atomic/help.md /help.md
COPY licenses /licenses

# Have to install glibc-langpack-en on UBI8 otherwise en_US.utf-8 does not exist
RUN if [ "$BASEOS" = "ubi8" ] ; then \
        ${PACKAGER} -y update --nodocs \
        && ${PACKAGER} -y install --nodocs \
                glibc-langpack-en ; \
fi

# Temporary HACK to disable postgres module
RUN if [ "$BASEOS" = "ubi8" ] ; then \
        echo "[postgresql]"  >> /etc/dnf/modules.d/postgresql.module \
        && echo "name=postgresql"  >> /etc/dnf/modules.d/postgresql.module \
        && echo "stream=10"  >> /etc/dnf/modules.d/postgresql.module \
        && echo "profiles="  >> /etc/dnf/modules.d/postgresql.module \
        && echo "state=disabled"  >> /etc/dnf/modules.d/postgresql.module ; \
fi

ENV LC_ALL en_US.utf-8
ENV LANG en_US.utf-8

ARG EPEL8_RPM=https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

RUN if [ "$BASEOS" = "ubi8" ] ; then \
        rpm -ivh ${EPEL8_RPM} \
        && ${PACKAGER} -y install --nodocs \
                bind-utils \
                gettext \
                hostname \
                procps-ng \
                nss_wrapper \
                less \
                vim-minimal \
        && ${PACKAGER} reinstall tzdata -y --nodocs \
        && sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/epel*.repo \
        && ${PACKAGER} -y clean all ; \
fi

# Add IvorySQL yum repository
RUN echo "[ivorysql4]"  >> /etc/yum.repos.d/ivorysql.repo \
        && echo "name=IvorySQL Server 4 \$releasever - \$basearch"  >> /etc/yum.repos.d/ivorysql.repo \
        && echo "baseurl=https://yum.highgo.com/dists/ivorysql-rpms/4/redhat/rhel-\$releasever-\$basearch"  >> /etc/yum.repos.d/ivorysql.repo \
        && echo "enabled=1"  >> /etc/yum.repos.d/ivorysql.repo \
        && echo "gpgcheck=0"  >> /etc/yum.repos.d/ivorysql.repo;