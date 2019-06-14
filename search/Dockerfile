ARG SEARCH_TAG
FROM alfresco/alfresco-search-services:${SEARCH_TAG}

# Configuration arguments passed from Docker Compose service definition
ARG SOLR_HOSTNAME
ARG ENABLE_SHARDING
ARG NUM_SHARDS
ARG SHARD_ID
ENV SOLR_HOSTNAME $SOLR_HOSTNAME
ENV ENABLE_SHARDING $ENABLE_SHARDING
ENV NUM_SHARDS $NUM_SHARDS
ENV SHARD_ID $SHARD_ID

# Base Docker Image for alfresco-search-services is exposing "$DIST_DIR/solrhome" as volume.
# This makes contents inside this folder immutable, so the files inside can't be modified
# in Dockerfile extensions. Because of this, "search_config_setup.sh" script is used in this
# Dockerfile to modify files under the folder, as the script is executed once the Container
# starts and changes are persisted.

# When Alfresco Search Services creates "alfresco" and "archive" SOLR Cores, configuration
# is copied from "rerank" template. So making changes in this template, generates both cores
# with the same configuration. If you are using volumes with existing cores, the changes below
# must be applied to configuration files on each node (instead on the template itself)

# Configure SOLR cores to run in HTTP mode
RUN sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=https/alfresco.secureComms=none/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
# Configure Alfresco Service Name
    sed -i '/^bash.*/i sed -i "'"s/alfresco.host=localhost/alfresco.host=alfresco/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
# Set Hostname for this Node
    sed -i '/^bash.*/i sed -i "'"s/solr.host=localhost/solr.host=${SOLR_HOSTNAME}/g"'" ${DIST_DIR}/solrhome/conf/shared.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
# Set Port Number and Sharding ID for this Shard Service
    sed -i '/^bash.*/i echo "\nsolr.port.ssl=8983\nshard.count=${NUM_SHARDS}\nshard.instance=${SHARD_ID}" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
    sed -i '/^bash.*/i echo "\nalfresco.port=8080\nalfresco.port.ssl=8443\nalfresco.baseUrl=/alfresco" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh;
