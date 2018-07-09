### Create /etc/systemd/system/tomcat.service
```
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking
#Environment variables just don't want to work for me
#So we make service.sh file todo the work
RemainAfterExit=true

ExecStart=/home/<username>/tomcat/bin/service.sh start
ExecStop=/home/<username>/tomcat/bin/service.sh stop

User=<username>
Group=<groupname>

[Install]
WantedBy=multi-user.target

```

## Create the service.sh
```
#!/bin/sh

export JAVA_HOME=/usr/local/java/jdk1.8.0_172
#export JRE_HOME=/usr/local/java/jdk1.8.0_172/jre
export CATALINA_PID=/home/<username>/tomcat/bin/catalina_pid.txt
export CATALINA_HOME=/opt/apache-tomcat-vX.Y.Z
export CATALINA_BASE=/home/<username>/tomcat
#export CATALINE_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC
#must use single quotes here
export JAVA_OPTS='-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'


start() {
    echo -n $"Starting $prog: "
    ${CATALINA_HOME}/bin/startup.sh
    RETVAL=$?
    return $RETVAL
}

stop() {
    echo -n $"Stopping $prog: "
    ${CATALINA_HOME}/bin/shutdown.sh
    RETVAL=$?
}


# See how we were called.
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    *)
        echo $"Usage: $prog {start|stop|help}"
        RETVAL=2
esac

exit $RETVAL

```
