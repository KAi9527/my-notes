#!/bin/sh

set -e

  

# Start Backend in background

echo "hello!"

echo "Starting Backend on 8081..."

# 创建日志目录

mkdir -p /app/logs /app/temp

touch /app/logs/be.log

# 沙盒后端服务默认使用8081端口

# 同时捕获 stdout 和 stderr 到日志文件

JAVA_OPTS="$JAVA_OPTS -server -Xss256K -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=52001 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.security.egd=file:/dev/./urandom -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector -Dlog4j2.AsyncQueueFullPolicy=Discard -Dlog4j2.discardThreshold=DEBUG -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:MetaspaceSize=512m -XX:ParallelGCThreads=8 -XX:ConcGCThreads=4 -Xmx8192m -Xms8192m -XX:CICompilerCount=2 -XX:G1ConcRefinementThreads=4"

java $JAVA_OPTS \

-classpath "/app/dsm-shub/conf/:/app/dsm-shub/lib/*" \

-Dbasedir=/app/dsm-shub \

-Ddeploy.app.name=${JOYGEN_DEVTOOL_RUNCFG_JAVA_TEST_JDOS_APP_NAME-:jdos_fin-dsm-shub} \

-Ddeploy.app.id=${JOYGEN_DEVTOOL_RUNCFG_JAVA_TEST_JDOS_APP_ID-:1596931} \

-Dserver.port=${SERVER_PORT:-8081} \

-Duser.home=${USER_HOME:-/app} \

com.jd.shub.web.ShubApplication >> /app/logs/be.log 2>&1 &

  

# # Start Nginx -> openresty

echo "Starting openresty on port 8080..."

openresty -g 'daemon off;'