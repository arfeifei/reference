# Apache Kafka Installtion

## Installation
```sh
cd /opt
wget http://apache-mirror.rbc.ru/pub/apache/kafka/0.10.1.0/kafka_2.11-0.10.1.0.tgz
tar xvzf kafka_2.11-0.10.1.0.tgz
ln -s kafka_2.11-0.10.1.0/ kafka
```

## Config ZooKeeper service for Kafaka
```sh
vi /etc/systemd/system/kafka-zookeeper.service
```
**kafka-zookeeper.service**
```ini
[Unit]
Description=Apache Zookeeper server (Kafka)
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=nano
Group=nano
Environment=JAVA_HOME=/etc/alternatives/java_sdk_1.8.0
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh

[Install]
WantedBy=multi-user.target
```
## Config Kafaka service
```sh
vi /etc/systemd/system/kafka.service
```
**kafka.service**
```ini
[Unit]
Description=Apache Kafka server (broker)
Documentation=http://kafka.apache.org/documentation.html
Requires=network.target remote-fs.target
After=network.target remote-fs.target kafka-zookeeper.service

[Service]
Type=simple
User=nano
Group=nano
Environment=JAVA_HOME=/etc/alternatives/java_sdk_1.8.0
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```
## Config Kafaka server on each node
```sh
vi kafka/config/server.properties
```
**Edit listeners properties**
```ini
listeners=PLAINTEXT://kafakaNodeServer:9092
```

## Config linux Bootup services
```sh
systemctl daemon-reload
systemctl start kafka-zookeeper.service
systemctl start kafka.service
```
