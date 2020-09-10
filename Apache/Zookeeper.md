
# Appache Zookeeper Install

## Prepare source

### As root
```sh
adduser zookeeper
passwd zookeeper
usermod -a -G wheel zookeeper
```

### As zookeeper
```sh
wget [http://apache.mirror.vexxhost.com/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz](#http://apache.mirror.vexxhost.com/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz)
tar -xzvf zookeeper-3.4.14.tar.gz

sudo ln -s /home/zookeeper/zookeeper-3.4.14 /opt/zookeeper
sudo mkdir /var/lib/zookeeper
sudo chown -R zookeeper:zookeeper /var/lib/zookeeper/

echo 1 > /var/lib/zookeeper/myid
cd /opt/zookeeper/conf/
cp zoo_sample.cfg zoo.cfg
# zoo.cfg content
# dataDir=/var/lib/zookeeper
```

### As root
```sh
vim /etc/systemd/system/zookeeper.service
systemctl daemon-reload
systemctl start zookeeper
systemctl enable zookeeper

```
zookeeper.service
```ini
[Unit]
Description=Apacke Zookeeper Daemon
Documentation=http://zookeeper.apache.org
#After=network.target remote-fs.target
#Requires=network.target remote-fs.target
Wants=syslog.target

[Service]
Type=forking
User=zookeeper
WorkingDirectory=/opt/zookeeper
ExecStart=/bin/bash /opt/zookeeper/bin/zkServer.sh start
ExecStop=/bin/bash /opt/zookeeper/bin/zkServer.sh stop
TimeoutSec=30
RestartSec=3
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

