ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG IVY_FULL
ARG BASE_IMAGE_NAME
ARG IVYO_VER
FROM ${BASE_IMAGE_NAME}:${BASEOS}-${IVY_FULL}-${IVYO_VER}-${BASEVER}

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG BASEOS
ARG PACKAGER

LABEL name="highgo-base-ext" \
	summary="Includes base extensions required to load in additional PostgreSQL extensions." \
	description="Includes base extensions required to load in additional PostgreSQL extensions."
