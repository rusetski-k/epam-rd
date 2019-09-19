# Homework report for 19.09.2019

## Scripts

### install.sh

```sh
#!/bin/bash

# install default openjdk
yum -y install java-11-openjdk

# download and install Oracle JDK
yum -y install wget cronie
wget -O - --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" https://download.oracle.com/otn-pub/java/jdk/12.0.2+10/e482c34c86bd4bf8b56c0b35558996b9/jdk-12.0.2_linux-x64_bin.tar.gz| tar xz -C /opt

# set up alternatives; yum install will add the first java alternative
alternatives --install /usr/bin/java java /opt/jdk-12.0.2/bin/java 2

# set first (openjdk) alternative as default 
echo 1 | alternatives --config java 

# install and enable epel-release repo
yum -y install epel-release

# install htop
yum -y install htop

# docker-ce install
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-18.06.1.ce-3.el7

# enable docker service
systemctl enable docker
systemctl start docker

# add cron job while preserving existing jobs
crontab - <<EOF
$(crontab -l)
5,30 12-15 * * 1 cd /tmp && rm -rf *.zip
EOF

# mongo d/l and install
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.0.tgz -O - | tar xz -C /opt
mkdir -p /data/db

# create service unit file
cat <<EOF >/etc/systemd/system/mongod.service
[Unit]
Description=MongoDB Database Service
Wants=network.target
After=network.target

[Service]
Type=forking
PIDFile=/var/run/mongodb/mongod.pid
ExecStart=/opt/mongodb-linux-x86_64-rhel70-4.2.0/bin/mongod --fork --pidfilepath /var/run/mongodb/mongod.pid --syslog
ExecStop=/bin/kill -TERM $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload

```

## Result verification

### Java version alternatives

#### OpenJDK
```
[root@74d0c4cff635 day4]# echo 1 | alternatives --config java &>/dev/null
[root@74d0c4cff635 day4]# java -version
openjdk version "11.0.4" 2019-07-16 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.4+11-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.4+11-LTS, mixed mode, sharing)
```
#### Oracle JDK
```
[root@74d0c4cff635 day4]# echo 2 | alternatives --config java &>/dev/null
[root@74d0c4cff635 day4]# java -version
java version "12.0.2" 2019-07-16
Java(TM) SE Runtime Environment (build 12.0.2+10)
Java HotSpot(TM) 64-Bit Server VM (build 12.0.2+10, mixed mode, sharing)
```


