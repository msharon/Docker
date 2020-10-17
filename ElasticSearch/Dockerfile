FROM @fromImage@

#USER esuser

# Source: https://github.com/pires/docker-elasticsearch/blob/master/Dockerfile
# Export HTTP & Transport
EXPOSE 9200 9300 9091 

# Set local variables 
ARG imageName=elasticsearch 
ARG imageHome=/home/${imageName}
ARG appHome=/home/${imageName}
ARG imageVersion=${image.version}
ARG yumRepoName=rhel-7-server-rpms
ARG imageTarName=${imageName}-${imageVersion}.tar
ARG registry=https://registry.dockerhub.io
ARG yumRepoDir=/etc/yum.repos.d/


ADD ${yumRepoName}.repo ${yumRepoDir}



# Install OS prerequisites
RUN microdnf install -y shadow-utils sudo wget unzip which hostname tar gzip java-1.8.0-openjdk-headless && \
    microdnf clean all && \
    mkdir -p ${appHome}/config/scripts ${appHome}/plugins ${appHome}/data/

ENV ES_VERSION=${imageVersion} \
                ES_NAME=${imageName} \ 
                ES_HOME=${imageHome}

ADD config ${appHome}/config/

COPY run.sh ${appHome} 

RUN cd /tmp && \
  #  useradd -m -d ${imageHome} -s /bin/bash ${imageName} && \
    curl -O ${registry}/${imageVersion}/${imageTarName} && \
    tar -xf ${imageTarName} && \    
    mv ${imageName}-${imageVersion} ${appHome} && \
    rm -rf /${imageHome}/${imageTarName} && \    
  #  mkdir -p ${appHome}/config/scripts ${appHome}/plugins && \
    cd ${appHome} && \ 
    curl -O ${registry}/plugins/${transportPluginName}/${transportPluginVersion}/${transportPluginFullName} && \
    mv ${transportPluginFullName} ${imageName}-${imageVersion}/
    

RUN groupadd elasticsearch

RUN useradd -u 1000 -r -g ${imageName} -m -d ${imageHome} -s /bin/bash ${imageName} && \
    chown -R ${imageName}:${imageName} ${appHome} && \ 
    chown -R ${imageName}:${imageName} /etc && \ 
    chown -R ${imageName}:${imageName} /var/lib && \
    chown -R ${imageName}:${imageName} ${appHome}  && \
    chown -R ${imageName}:${imageName} ${appHome}/data/ && \
    chmod -R 777 ${imageHome} && \
    chmod -R 777 ${appHome}  && \
    chmod -R 777 ${appHome}/run.sh && \
    chmod -R 777 ${appHome}/data/ && \
    chmod -R 777 ${appHome}/config/ && \ 
    chmod -R 777 ${appHome}/plugins/ && \
    chmod -R 777 ${appHome}/elasticsearch-6.6.0/ && \
    echo $'elasticsearch ALL= NOPASSWD:SETENV: /usr/bin/yum install* elasticsearch*,/usr/bin/yum remove* elasticsearch*,/usr/bin/yum update* elasticsearch*,/usr/bin/yum downgrade* elasticsearch*,/usr/bin/systemctl start elasticsearch*,/usr/bin/systemctl stop elasticsearch*,/usr/bin/systemctl restart elasticsearch*,/usr/bin/systemctl enable elasticsearch*,/usr/bin/systemctl disable elasticsearch*,/usr/bin/su - /usr/share/elasticsearch/plugins/*,/usr/bin/su - /usr/share/elasticsearch/bin/x-pack/*,/usr/bin/sh *,/usr/bin/chown *,/usr/bin/chmod *' >> /etc/sudoers
    
RUN chown root:root /etc/sudoers /etc/sudoers.d -R 
    
USER ${imageName}    

WORKDIR ${imageHome}
RUN rm -rf elasticsearch-6.6.0/config elasticsearch-6.6.0/plugins
RUN mv config plugins elasticsearch-6.6.0/


ENV PATH ${appHome}/bin:$PATH
ENV PATH ${appHome}:$PATH

WORKDIR ${imageHome}

# Copy configuration
# COPY config ${appHome}/config

# Copy run script

# Volume for Elasticsearch data
VOLUME ["/home/elasticsearch/data"]

# Set environment variables defaults
ENV ES_JAVA_OPTS="-Xms512m -Xmx512m" \
 CLUSTER_NAME=${imageName}-default \
 NODE_MASTER=true \
 NODE_DATA=true \
 NODE_INGEST=true \
 HTTP_ENABLE=true \
 NETWORK_HOST=_site_ \
 HTTP_CORS_ENABLE=true \
 HTTP_CORS_ALLOW_ORIGIN=* \
 NUMBER_OF_MASTERS=1 \
 MAX_LOCAL_STORAGE_NODES=1 \
 SHARD_ALLOCATION_AWARENESS="" \
 SHARD_ALLOCATION_AWARENESS_ATTR="" 


# Source: https://github.com/pires/docker-elasticsearch-kubernetes/blob/master/Dockerfile
ENV DISCOVERY_SERVICE ${imageName}-service

# Enable memory lock
ENV MEMORY_LOCK true

CMD ["/home/elasticsearch/run.sh"]
