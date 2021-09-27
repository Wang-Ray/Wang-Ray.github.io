---
layout: post
title: "实践Java应用启动和停止shell脚本"
date: 2018-12-26 13:53:00 +0800
categories: OS
tags: linux find
---



```shell
#!/bin/bash

source ~/.bashrc
BASEPATH=$(cd `dirname $0`; pwd)

PROG_NAME=$0
ACTION=$1
APP_NAME=sample-service

# not prod
JVM_DEBUG_OPTS="-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8686"
JAVA_OPTS="-server -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+AlwaysPreTouch -XX:AutoBoxCacheMax=20000 -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+HeapDumpOnOutOfMemoryError"
# prod
if [ -z $ENV ]
then
    export SPRING_PROFILES_ACTIVE=prod
    export ENV=pro
    export APOLLO_META=http://172.19.177.117
    JVM_DEBUG_OPTS=""
    JAVA_OPTS="-server -Xms4G -Xmx4G -Xmn2G -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+AlwaysPreTouch -XX:AutoBoxCacheMax=20000 -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+HeapDumpOnOutOfMemoryError"
fi

usage() {
    echo "Usage: $PROG_NAME {start|stop|restart}"
    exit 2
}

start_application() {
    # check java process
    checkjavapid=`ps -ef | grep java | grep ${APP_NAME} | grep -v grep |grep -v 'deploy.sh'| awk '{print$2}'`
    if [[ $checkjavapid ]];then
        echo -e  "java process exists"
        return
    fi

    echo "starting java process"
    nohup java $JAVA_OPTS $JVM_DEBUG_OPTS -jar BASEPATH/*.jar >/dev/null 2>&1 &
    echo "started java process"
}

stop_application() {
    # check java process
    checkjavapid=`ps -ef | grep java | grep ${APP_NAME} | grep -v grep |grep -v 'deploy.sh'| awk '{print$2}'`
    if [[ ! $checkjavapid ]];then
        echo -e "\rno java process"
        return
    fi

    echo "stop java process"
    times=60
    for e in $(seq $times)
    do
        COSTTIME=$(($times - $e ))
        checkjavapid=`ps -ef | grep java | grep ${APP_NAME} | grep -v grep |grep -v 'deploy.sh'| awk '{print$2}'`
        if [[ $checkjavapid ]];then
	    if [[ $COSTTIME -gt 0 ]];then
                kill $checkjavapid
	    else
                kill -9 $checkjavapid
		sleep 1
                echo -e "\rjava process has exited forcedly"
                break;
	    fi
            sleep 1
            echo -e  "\r        -- stopping java lasts `expr $COSTTIME` seconds."
        else
            echo -e "\rjava process has exited normally"
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

