# OMAR WMTS

## Purpose

The OMAR WMTS application provides OGC capabilities for the WMTS standard, serving out image chips from O2's raster data holdings.

## Installation in Openshift

**Assumption:** The omar-wmts-app docker image is pushed into the OpenShift server's internal docker registry and available to the project.

### Environment variables

|Variable|Value|
|------|------|
|SPRING_PROFILES_ACTIVE|Comma separated profile tags (*e.g. production, dev*)|
|SPRING_CLOUD_CONFIG_LABEL|The Git branch from which to pull config files (*e.g. master*)|
