FROM registry.access.redhat.com/ubi8/ubi
MAINTAINER Alexey Pustovalov <alexey.pustovalov@zabbix.com>

ARG MAJOR_VERSION=5.0
ARG RELEASE=15
ARG ZBX_VERSION=${MAJOR_VERSION}.15

ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git
ENV TERM=xterm ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES} \
    MIBDIRS=/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL
ENV TINI_VERSION=v0.19.0

LABEL name="zabbix/zabbix-proxy-sqlite-50" \
      maintainer="alexey.pustovalov@zabbix.com" \
      vendor="Zabbix LLC" \
      version="${MAJOR_VERSION}" \
      release="${RELEASE}" \
      summary="Zabbix proxy (SQLite3)" \
      description="Zabbix proxy with SQLite3 database support" \
      url="https://www.zabbix.com/" \
      run="docker run --name zabbix-proxy -p 10051:10051 -d registry.connect.redhat.com/zabbix/zabbix-proxy-sqlite-50:${ZBX_VERSION}" \
      io.k8s.description="Zabbix proxy with SQLite3 database support" \
      io.k8s.display-name="Zabbix proxy (SQLite3)" \
      io.openshift.expose-services="10051:10051" \
      io.openshift.tags="zabbix,zabbix-proxy,sqlite" \
      org.label-schema.name="zabbix-proxy-sqlite-rhel" \
      org.label-schema.vendor="Zabbix LLC" \
      org.label-schema.url="https://zabbix.com/" \
      org.label-schema.description="Zabbix proxy with SQLite3 database support" \
      org.label-schema.vcs-ref="${VCS_REF}" \
      org.label-schema.build-date="${BUILD_DATE}" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.license="GPL v2.0" \
      org.label-schema.usage="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.label-schema.version="${ZBX_VERSION}" \
      org.label-schema.vcs-url="${ZBX_SOURCES}" \
      org.label-schema.docker.cmd="docker run --name zabbix-proxy -p 10051:10051 -d registry.connect.redhat.com/zabbix/zabbix-proxy-sqlite3-50:${ZBX_VERSION}"

STOPSIGNAL SIGTERM

COPY ["licenses", "/licenses"]

RUN set -eux && INSTALL_PKGS="bash \
            fping \
            iputils \
            libcurl \
            libevent \
            openldap \
            libssh \
            libxml2 \
            net-snmp-agent-libs \
            OpenIPMI-libs \
            pcre \
            sqlite-libs \
            unixODBC" && \
    REPOLIST="rhel-8-for-x86_64-baseos-rpms,rhel-8-for-x86_64-appstream-rpms,epel" && \
    dnf -y update-minimal --disablerepo "*" --enablerepo ubi-8-baseos --setopt=tsflags=nodocs \
        --security --sec-severity=Important --sec-severity=Critical && \
    dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm && \
    dnf -y install --disablerepo "*" --enablerepo "${REPOLIST}" --setopt=install_weak_deps=False --best \
        --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    groupadd --system --gid 1995 zabbix && \
    useradd \
            --system --comment "Zabbix monitoring system" \
            -g zabbix -G root \
            --uid 1997 \
            --shell /sbin/nologin \
            --home-dir /var/lib/zabbix/ \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /var/lib/zabbix/db_data && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    curl -L "https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini" -o /sbin/tini && \
    curl -L "https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini.asc" -o /tmp/tini.asc && \
    export GNUPGHOME="$(mktemp -d)" && \
    for server in $(shuf -e ha.pool.sks-keyservers.net \
                            hkp://p80.pool.sks-keyservers.net:80 \
                            ipv4.pool.sks-keyservers.net \
                            keyserver.ubuntu.com \
                            keyserver.pgp.com \
                            pgp.mit.edu) ; do \
        gpg --keyserver "$server" --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 && break || : ; \
    done && \
    gpg --batch --verify /tmp/tini.asc /sbin/tini && \
    rm -r "$GNUPGHOME" /tmp/tini.asc && \
    chmod +x /sbin/tini && \
    dnf -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki

RUN set -eux && INSTALL_PKGS="autoconf \
            automake \
            libcurl-devel \
            libevent-devel \
            libssh-devel \
            libxml2-devel \
            sqlite-devel \
            net-snmp-devel \
            openldap-devel \
            pcre-devel \
            git \
            gcc \
            make \
            unixODBC-devel" && \
    REPOLIST="rhel-8-for-x86_64-baseos-rpms,rhel-8-for-x86_64-appstream-rpms" && \
    dnf -y install --disablerepo "*" --enablerepo "${REPOLIST}" --setopt=install_weak_deps=False --best \
        --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    cd /tmp/ && \
    git -c advice.detachedHead=false clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    cd /tmp/zabbix-${ZBX_VERSION} && \
    zabbix_revision=`git rev-parse --short HEAD` && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" include/version.h && \
    ./bootstrap.sh && \
    export CFLAGS="-fPIC -pie -Wl,-z,relro -Wl,-z,now" && \
    ./configure \
            --datadir=/usr/lib \
            --libdir=/usr/lib/zabbix \
            --sysconfdir=/etc/zabbix \
            --prefix=/usr \
            --enable-agent \
            --enable-proxy \
            --with-sqlite3 \
            --with-ldap \
            --with-libcurl \
            --with-libxml2 \
            --with-net-snmp \
#            --with-openipmi \
            --with-openssl \
            --with-ssh \
            --with-unixodbc \
            --enable-ipv6 \
            --silent && \
    make -j"$(nproc)" -s dbschema && \
    make -j"$(nproc)" -s && \
    cp src/zabbix_proxy/zabbix_proxy /usr/sbin/zabbix_proxy && \
    cp src/zabbix_get/zabbix_get /usr/bin/zabbix_get && \
    cp src/zabbix_sender/zabbix_sender /usr/bin/zabbix_sender && \
    cp conf/zabbix_proxy.conf /etc/zabbix/zabbix_proxy.conf && \
    chown --quiet -R zabbix:root /etc/zabbix && \
    cd /tmp/ && \
    rm -rf /tmp/zabbix-${ZBX_VERSION}/ && \
    dnf -y history undo last && \
    dnf -y clean all && \
    chown --quiet -R zabbix:root /etc/zabbix/ /var/lib/zabbix/ && \
    chgrp -R 0 /etc/zabbix/ /var/lib/zabbix/ && \
    chmod -R g=u /etc/zabbix/ /var/lib/zabbix/ && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki

EXPOSE 10051/TCP

WORKDIR /var/lib/zabbix

VOLUME ["/var/lib/zabbix/snmptraps"]

COPY ["docker-entrypoint.sh", "/usr/bin/"]

ENTRYPOINT ["/sbin/tini", "--", "/usr/bin/docker-entrypoint.sh"]

USER 1997

CMD ["/usr/sbin/zabbix_proxy", "--foreground", "-c", "/etc/zabbix/zabbix_proxy.conf"]
