FROM ubuntu:22.04
ENV ZK_USER=zookeeper \
ZK_USER_ID=1010 \
ZK_GROUP_ID=1010 \
ZK_DATA_DIR=/var/lib/zookeeper/data \
ZK_DATA_LOG_DIR=/var/lib/zookeeper/log \
ZK_LOG_DIR=/var/log/zookeeper \
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

ARG ZK_DIST=zookeeper-3.8.2
ENV ZK_DIST_APACHE=apache-${ZK_DIST}-bin
ENV ZK_DIST_PACKAGE=${ZK_DIST_APACHE}.tar.gz
RUN set -x \
    && apt-get update \
    && apt-get install -y openjdk-11-jre-headless wget netcat-openbsd \
    && wget -q "http://www.apache.org/dist/zookeeper/${ZK_DIST}/${ZK_DIST_PACKAGE}" \
    && tar -xzf "${ZK_DIST_PACKAGE}" -C /opt \
    && rm -r "${ZK_DIST_PACKAGE}" \
    && ln -s /opt/${ZK_DIST_APACHE} /opt/zookeeper \
    && rm -rf /opt/zookeeper/README.txt \
    /opt/zookeeper/NOTICE.txt \
    /opt/zookeeper/README_packaging.txt \
    /opt/zookeeper/docs \
    && apt-get autoremove -y wget \
    && rm -rf /var/lib/apt/lists/*

# Copy configuration generator script to bin
COPY scripts /opt/zookeeper/bin/

# Create a user for the zookeeper process and configure file system ownership 
# for nessecary directories and symlink the distribution as a user executable
RUN set -x \
	&& groupadd -g $ZK_GROUP_ID $ZK_USER \
	&& useradd $ZK_USER -u $ZK_USER_ID -g $ZK_USER \
    && [ `id -u $ZK_USER` -eq $ZK_USER_ID ] \
    && [ `id -g $ZK_USER` -eq $ZK_GROUP_ID ] \
    && mkdir -p $ZK_DATA_DIR $ZK_DATA_LOG_DIR $ZK_LOG_DIR /usr/share/zookeeper /tmp/zookeeper /usr/etc/ \
	&& chown -R "$ZK_USER:$ZK_USER" /opt/${ZK_DIST_APACHE} $ZK_DATA_DIR $ZK_LOG_DIR $ZK_DATA_LOG_DIR /tmp/zookeeper \
	&& ln -s /opt/zookeeper/conf/ /usr/etc/zookeeper \
	&& ln -s /opt/zookeeper/bin/* /usr/bin \
	&& ln -s /opt/zookeeper/lib/* /usr/share/zookeeper 
