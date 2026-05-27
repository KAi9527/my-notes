# Build and Runtime Stage
FROM joycode-cn-north-1.jcr.service.jdcloud.com/joygen/maven:3.8.6-jdk-8-amd AS build
WORKDIR /app

ENV MAVEN_OPTS="-Dmaven.repo.local=/app/.m2/repository"
ENV MAVEN_CLI_OPTS="-Dstyle.color=never -DtrimStackTrace=false --batch-mode --errors --fail-at-end --show-version -DinstallAtEnd=true -DdeployAtEnd=true"

# The public Maven mirror cannot resolve JD-patched versions in this POM.
# Override them during Docker builds without changing source pom.xml.
ARG LOG4J2_VERSION=2.18.0
ARG LOG4J_VERSION=2.17.1
ARG FASTJSON_VERSION=1.2.83
ARG DEBIAN_MIRROR=https://mirrors.aliyun.com/debian
ARG DEBIAN_SECURITY_MIRROR=https://mirrors.aliyun.com/debian-security

ENV JAVA_OPTS="-XX:+UseG1GC -Dfile.encoding=UTF-8"
ENV SERVER_PORT=8081
ENV SPRING_PROFILES_ACTIVE=docker
ENV LOG_PATH=/app/logs
ENV JDOS_ENV=dev
ENV RUNTIME_CONFIG_ADDRESS=6.120.67.34

# 复用缓存，避免每次重复运行
RUN set -eux; \
    sed -i \
        -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
        -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
        /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends wget gnupg ca-certificates && \
    wget -O - https://openresty.org/package/pubkey.gpg | apt-key add - && \
    codename="$(. /etc/os-release && echo "$VERSION_CODENAME")" && \
    arch="$(dpkg --print-architecture)" && \
    if [ "$arch" = "arm64" ]; then \
        echo "deb http://openresty.org/package/arm64/debian $codename openresty" > /etc/apt/sources.list.d/openresty.list; \
    else \
        echo "deb http://openresty.org/package/debian $codename openresty" > /etc/apt/sources.list.d/openresty.list; \
    fi && \
    apt-get update && \
    apt-get install -y --no-install-recommends openresty && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY . .
RUN ls -l .
RUN --mount=type=cache,id=dsm-shub-m2,target=/app/.m2/repository \
    mvn -s ./mysettings.xml clean package -DskipTests -Dmaven.test.skip=true && \
    ZIP_FILE="$(find /app/dsm-shub-web/target -maxdepth 1 -type f -name '*-dist.zip' -print -quit)" && \
    test -n "${ZIP_FILE}" && \
    mkdir -p /app/dsm-shub /app/logs /app/temp && \
    unzip -o -q "${ZIP_FILE}" -d /app/dsm-shub

# Copy Nginx/openresty Config
COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
# Copy Start Script
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh

EXPOSE 8080
EXPOSE 8081
VOLUME ["/app/logs", "/app/temp"]

CMD ["/app/start.sh"]
