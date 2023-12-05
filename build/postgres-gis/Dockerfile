ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
FROM ${PREFIX}/highgo-postgres-gis-base:${BASEOS}-${PG_FULL}-${BASEVER} as step1

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG BASEOS
ARG PACKAGER

LABEL name="postgres-gis" \
	summary="Includes PostGIS extensions on top of highgo-ivory" \
	description="An identical image of highgo-ivory with the extra PostGIS packages added for users that require PostGIS." \
	io.k8s.description="postgres-gis container" \
	io.k8s.display-name="Highgo PostGIS" \
	io.openshift.tags="postgresql,postgres,postgis,spatial,geospatial,gis,map,database,ha,highgo,ivory,ivorysql"

USER 0

RUN if [ "$BASEOS" = "ubi8" ] ; then \
	${PACKAGER} -y install gcc gcc-c++; \
fi

#build postgis from source
#1.install proj
ADD build/postgres-gis/sqlite-autoconf-3400000.tar.gz build/postgres-gis/proj-8.2.1.tar.gz /
RUN cd sqlite-autoconf-3400000 \
	&& sed '1i\#define SQLITE_ENABLE_COLUMN_METADATA 1' sqlite3.c \
	&& ./configure --prefix=/usr/local/sqlite \
	&& make \
	&& make install \
	&& ln -s /usr/local/sqlite/bin/sqlite3   /usr/bin/sqlite3 \
	&& sqlite3 -version
ENV PKG_CONFIG_PATH=/usr/local/sqlite/lib/pkgconfig:$PKG_CONFIG_PATH
RUN ${PACKAGER} install -y libtiff libtiff-devel.x86_64 libcurl-devel.x86_64
RUN cd proj-8.2.1 \
	&& ./configure --prefix=/usr/local/proj-8.2.1 \
	&& make \
	&& make install
#2.install gdal
RUN ${PACKAGER} install -y automake
ADD build/postgres-gis/gdal-3.4.3.tar.gz /
RUN cd /gdal-3.4.3 \
	&& sh autogen.sh \
	&& ./configure  --prefix=/usr/local/gdal-3.4.3 --with-proj=/usr/local/proj-8.2.1 \
	&& make \
	&& make install
#3.install other deps
RUN if [ "$BASEOS" = "centos8" ] ; then \
	${PACKAGER} config-manager --set-enabled powertools \
	&& ${PACKAGER} install -y which libtool libxml2-devel geos-devel protobuf-c-devel ; \
fi

ADD build/postgres-gis/geos-3.9.2.tar.bz2 /
ADD build/postgres-gis/protobuf-c-1.4.1.tar.gz /
ADD build/postgres-gis/protobuf-3.20.1.tar.gz /
RUN if [ "$BASEOS" = "ubi8" ] ; then \
	cd geos-3.9.2 \
	&& ./configure --prefix=/usr/local/geos-3.9.2 \
	&& make && make install; \
fi
RUN if [ "$BASEOS" = "ubi8" ] ; then \
	${PACKAGER} -y install libtool \
        && cd /protobuf-3.20.1 \
        && sh autogen.sh \
        && ./configure  --prefix=/usr/local/protobuf-3.20.1 \
        && make && make install \
        && export PROTOBUF_HOME=/usr/local/protobuf-3.20.1 \
        && export PATH=$PROTOBUF_HOME/bin:$PATH \
        && export PKG_CONFIG_PATH=$PROTOBUF_HOME/lib/pkgconfig:$PKG_CONFIG_PATH; \
fi
RUN if [ "$BASEOS" = "ubi8" ] ; then \
        cd /protobuf-c-1.4.1 \
        && export PROTOBUF_HOME=/usr/local/protobuf-3.20.1 \
        && export PATH=$PROTOBUF_HOME/bin:$PATH \
        && export PKG_CONFIG_PATH=$PROTOBUF_HOME/lib/pkgconfig:$PKG_CONFIG_PATH \
        && ./configure --prefix=/usr/local/protobuf-c-1.4.1 \
        && make && make install \
        && export PROTOBUFC_HOME=/usr/local/protobuf-c-1.4.1 \
        && export PATH=$PROTOBUF_HOME/bin:$PROTOBUFC_HOME/bin:$PATH \
        && export PKG_CONFIG_PATH=$PROTOBUFC_HOME/lib:$PKG_CONFIG_PATH; \
fi
RUN if [ "$BASEOS" = "ubi8" ] ; then \
	${PACKAGER} -y install which libtool libxml2-devel redhat-rpm-config clang llvm; \
fi

ADD build/postgres-gis/postgis-3.4.0.tar.gz /
RUN cd postgis-3.4.0 \
        && export PROTOBUFC_HOME=/usr/local/protobuf-c-1.4.1 \
        && export PATH=$PROTOBUF_HOME/bin:$PROTOBUFC_HOME/bin:$PATH \
        && export PKG_CONFIG_PATH=$PROTOBUFC_HOME/lib:$PKG_CONFIG_PATH \
	&& sh autogen.sh \
	&& ./configure  --with-geosconfig=/usr/local/geos-3.9.2/bin/geos-config --with-projdir=/usr/local/proj-8.2.1 --with-gdalconfig=/usr/local/gdal-3.4.3/bin/gdal-config --with-protobufdir=/usr/local/protobuf-c-1.4.1 \
	&& make \
	&& make install

##############################################################################
##### stage2
##############################################################################

FROM ${PREFIX}/highgo-postgres-gis-base:${BASEOS}-${PG_FULL}-${BASEVER}
ARG BASEOS
ARG PACKAGER

LABEL name="postgres-gis" \
        summary="Includes PostGIS extensions on top of highgo-ivory" \
        description="An identical image of highgo-ivory with the extra PostGIS packages added for users that require PostGIS." \
        io.k8s.description="postgres-gis container" \
        io.k8s.display-name="Highgo PostGIS" \
        io.openshift.tags="postgresql,postgres,postgis,spatial,geospatial,gis,map,database,ha,highgo,ivory,ivorysql"

USER 0
RUN if [ "$BASEOS" = "centos8" ] ; then \
	${PACKAGER} install -y libxml2 json-c geos protobuf protobuf-c libtiff  && ${PACKAGER} -y clean all ; \
fi
RUN if [ "$BASEOS" = "ubi8" ] ; then \
        ${PACKAGER} install -y geos311 libtiff  && ${PACKAGER} -y clean all ; \
fi
RUN rm -rf /usr/local/ivorysql/ivorysql-3
COPY --from=step1 /usr/local/proj-8.2.1/lib/*.so /usr/local/proj-8.2.1/lib/
COPY --from=step1 /usr/local/gdal-3.4.3/lib/*.so /usr/local/gdal-3.4.3/lib/
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/bin/pgsql2shp /usr/local/ivorysql/ivorysql-3/bin/pgsql2shp 
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/bin/raster2pgsql /usr/local/ivorysql/ivorysql-3/bin/raster2pgsql 
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/bin/shp2pgsql /usr/local/ivorysql/ivorysql-3/bin/shp2pgsql 
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/lib/bitcode /usr/local/ivorysql/ivorysql-3/lib/bitcode
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/lib/postgis* /usr/local/ivorysql/ivorysql-3/lib/
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/share/extension/postgis* /usr/local/ivorysql/ivorysql-3/share/extension/
#COPY --from=step1 /usr/local/ivorysql/ivorysql-3/share/contrib/postgis-3.2 /usr/local/ivorysql/ivorysql-3/share/contrib/postgis-3.2
COPY --from=step1 /usr/local/ivorysql/ivorysql-3 /usr/local/ivorysql/ivorysql-3

RUN sed -i '$a\/usr/local/proj-8.2.1/lib' /etc/ld.so.conf \
	&& sed -i '$a\/usr/local/gdal-3.4.3/lib' /etc/ld.so.conf \
	&& ldconfig -v

# open up the postgres port
EXPOSE 5432 5866 5333

ADD bin/postgres-gis /opt/crunchy/bin/postgres

ENTRYPOINT ["/opt/crunchy/bin/uid_postgres.sh"]

#USER 26
USER 998

CMD ["/opt/crunchy/bin/start.sh"]
