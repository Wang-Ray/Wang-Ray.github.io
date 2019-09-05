---
layout: post
title: "实践Java应用启动和停止shell脚本"
date: 2018-12-26 13:53:00 +0800
categories: Linux
tags: linux find
---



```shell
#!/bin/bash

PROG_NAME=$0
ACTION=$1
APP_NAME=sample

# not prod
JVM_DEBUG_OPTS="-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8686"
JAVA_OPTS="-server -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+AlwaysPreTouch -XX:AutoBoxCacheMax=20000 -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime"
# prod
if [ -z $ENV ]
then
    export SPRING_PROFILES_ACTIVE=prod
    export ENV=pro
    export APOLLO_META=http://192.168.1.100
    JVM_DEBUG_OPTS=""
    JAVA_OPTS="-server -Xms4G -Xmx4G -Xmn1800m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+AlwaysPreTouch -XX:AutoBoxCacheMax=20000 -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime"
fi

usage() {
    echo "Usage: $PROG_NAME {start|stop|restart}"
    exit 2
}

start_application() {
    echo "starting java process"
    nohup java $JAVA_OPTS $JVM_DEBUG_OPTS -jar *.jar >/dev/null 2>&1 &
    echo "started java process"
}

stop_application() {
   checkjavapid=`ps -ef | grep java | grep ${APP_NAME} | grep -v grep |grep -v 'deploy.sh'| awk '{print$2}'`
   
   if [[ ! $checkjavapid ]];then
      echo -e "\rno java process"
      return
   fi

   echo "stop java process"
   times=60
   for e in $(seq 60)
   do
        sleep 1
        COSTTIME=$(($times - $e ))
        checkjavapid=`ps -ef | grep java | grep ${APP_NAME} | grep -v grep |grep -v 'deploy.sh'| awk '{print$2}'`
        if [[ $checkjavapid ]];then
            kill $checkjavapid
            echo -e  "\r        -- stopping java lasts `expr $COSTTIME` seconds."
        else
            echo -e "\rjava process has exited"
            break;
        fi
   done
   echo ""
}
start() {
    start_application
}
stop() {
    stop_application
}
case "$ACTION" in
    start)
        start
    ;;
    stop)
        stop
    ;;
    restart)
        stop
        start
    ;;
    *)
        usage
    ;;
esac
```

