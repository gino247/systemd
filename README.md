# small systemd example
Note all systemd process run as root.

Rules:
* If systemd no see process it will shut it down. **RemainAfterExit=true**
* If installation is not creating service symbolic links, you forgot **WantedBy = multi-user.target**, under \[Install\] unit

Everything as **root**

### Create /etc/systemd/system/serviceTest.sh
```
[Unit]
Description = service test daemon
After = network.target

[Service]
#since we starting a separate process we fork
Type=forking
WorkingDirectory = /root
#since systemd is not monitoring the new process
RemainAfterExit=true
ExecStart = /root/serviceTest.sh start
ExecStop = /root/serviceTest.sh stop
ExecReload = /root/serviceTest.sh reload

[Install]
WantedBy = multi-user.target
```
**Everything don't have to be in /root, as long as root has access to the file**

### Create needed script file, serviceTest.sh
```
#! /bin/bash

function do_start ( )
{
        echo "Hi, start time $(date)" | tee -a /root/serviceTest.log
        sleep  5
}

function do_stop ( )
{
        echo "Bye, stopped time $(date)" | tee -a /root/serviceTest.log
}

function do_status ( )
{
        echo "Status, asked time $(date)" | tee -a /root/serviceTest.log
}

# Some Things That run always
touch  /var/lock/serviceTest

# Management instructions of the service
echo "Command: $0 $1" | tee -a /root/serviceTest.log
case  "$1"  in
        start )
                do_start
                ;;
        stop )
                do_stop
                ;;
        reload )
                do_stop
                sleep  1
                do_start
                ;;
        status )
                do_status
                ;;
        * )
        echo  "Usage: $0 {start | stop | reload | status}"
        exit  1
        ;;
esac

exit  0
```

### Make script runnable
```
chmod +x serviceTest.sh
```

### Install service
```
systemctl enable serviceTest.service
```
### Output should look like this
**Created symlink from /etc/systemd/system/multi-user.target.wants/serviceTest.service to /etc/systemd/system/serviceTest.service.**

### Start service
```
systemctl start serviceTest.service
```

### Stop service
```
systemctl stop serviceTest.service
```

### If you change the service file, serviceTest.service
```
systemctl deamon-reload
```

### Check what the service did
```
journal -xe
```
