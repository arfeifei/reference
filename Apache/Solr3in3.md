# Install Apache Solr as cluster mode on multiple server
## Prepare source

```sh
wget http://apache.forsale.plus/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

wget http://apache.forsale.plus/lucene/solr/7.1.0/solr-7.1.0.tgz
```

## Generate Self-signed certificate [Reference](https://lucene.apache.org/solr/guide/7_1/enabling-ssl.html)

Replace solr-#.mycompany.com with the real hostname so does to IP address. Can be done in one machine

``` sh
keytool -genkeypair -alias solr-ssl -keyalg RSA -keysize 2048 -keypass secret -storepass secret -validity 9999 -keystore solr-ssl.keystore.jks -ext SAN=DNS:solr-1.mycompany.com,DNS:solr-2.mycompany.com,DNS:solr-3.mycompany.com,DNS:localhost,IP:192.168.1.213,IP:192.168.1.163,IP:192.168.1.165,IP:127.0.0.1 -dname "CN=localhost, OU=Department, O=Company, L=Toronto, ST=Ontario, C=Canada"
```

## Install Zookeeper (on each node)

``` sh
tar xfvz zookeeper-3.4.10.tar.gz
sudo mv zookeeper-3.4.10 /opt/zookeeper
cd /var/lib
sudo mkdir zookeeper
cd /opt/zookeeper/conf
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
```

change the following config

```sh
dataDir=/var/lib/zookeeper
server.1=solr-1.mycompany.com:2222:3300
server.2=solr-2.mycompany.com:2222:3300
server.3=solr-3.mycompany.com:2222:3300
```

zookeeper as cluster mode. $(node_number) equals 1, 2, 3

``` sh
sudo chown -R solr:solr /var/lib/zookeeper/
echo $(node_number) > /var/lib/zookeeper/myid
cd /var/lib/zookeeper/
/opt/zookeeper/bin/zkServer.sh start
/opt/zookeeper/bin/zkServer.sh status
cd ~/downloads/
```

## Install Solr (on each node)

``` sh
tar xzf solr-7.1.0.tgz  solr-7.1.0/bin/install_solr_service.sh --strip-components=2
sudo bash ./install_solr_service.sh solr-7.1.0.tgz

# Installing symlink /opt/solr -> /opt/solr-7.1.0 ...
# Installing /etc/init.d/solr script ...
# Installing /etc/default/solr.in.sh ...
sudo chown -R solr:solr /opt/solr
sudo chown -R solr:solr /opt/solr-7.1.0
# put self-signed certificate on each solr node server etc
cp ~/solr-ssl.keystore.jks /opt/solr/server/etc
scp ~/solr-ssl.keystore.jks solr@solr-2.mycompany.com:/opt/solr/server/etc
scp ~/solr-ssl.keystore.jks solr@solr-3.mycompany.com:/opt/solr/server/etc

```

## Setup firewall (for Each node)

``` sh
sudo firewall-cmd --zone=public --add-port=2181/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8983/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2222-3300/tcp --permanent

sudo firewall-cmd --reload

sudo service solr stop
```

## Configure Solr.in.sh (for Each node)

``` sh
sudo vi /etc/default/solr.in.sh
#add
SOLR_HOST="$(hostname)"
SOLR_TIMEZONE="America/New_York"
SOLR_PORT=8983
SOLR_HEAP="1g"
SOLR_SSL_KEY_STORE=/opt/solr/server/etc/solr-ssl.keystore.jks
SOLR_SSL_KEY_STORE_PASSWORD=secret
SOLR_SSL_KEY_STORE_TYPE=JKS
SOLR_SSL_TRUST_STORE=/opt/solr/server/etc/solr-ssl.keystore.jks
SOLR_SSL_TRUST_STORE_PASSWORD=secret
SOLR_SSL_TRUST_STORE_TYPE=JKS
SOLR_SSL_NEED_CLIENT_AUTH=false
SOLR_SSL_WANT_CLIENT_AUTH=false
ZK_HOST="solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181"
# increase node pkiauth ttl from default 10 seconds to 60 seconds when clock out of synchronized
SOLR_OPTS="$SOLR_OPTS -Dpkiauth.ttl=60000"
```

Remove default solr.xml parepare for zookeeper

```sh
mv /var/solr/data/solr.xml  /var/solr/data/solr.xml.old
```

## Add sharedLib

``` sh
tar xvf opt.tar
cd opt
sudo mv shared-lib /opt
sudo chown -R solr:solr /opt/shared-lib/
```

## Create business configsets (on only one node)

``` sh
tar xvf configsets.tar
cd /opt/solr/server/scripts/cloud-scripts
chmod 755 *.sh
./zkcli.sh -zkhost solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181 -cmd putfile /solr.xml ~/downloads/configsets/solr.xml
./zkcli.sh -zkhost solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181 -cmd putfile /security.json ~/downloads/configsets/security.json
#Set all solr cluster use https insteadof http
./zkcli.sh -zkhost solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181 -cmd clusterprop -name urlScheme -val https
```

Example security.json content default solr admin as solr:SolrRocks

``` json
{
  "authentication":{
    "blockUnknown":true,
    "class":"solr.BasicAuthPlugin",
    "credentials":{
      "solr":"IV0EHq1OnNrj6gvRCwvFwTrZ1+z1oBbnQdiVC3otuq0= Ndd7LKvVBAaZIF0QAVi1ekCfAJXr1GGfLtRUXhgrF8c="}
    },
  "authorization":{
    "class":"solr.RuleBasedAuthorizationPlugin",
    "permissions":[
      {
        "name":"security-edit",
        "role":"admin"},
      {
        "name":"schema-edit",
        "role":"admin"},
      {
        "name":"config-edit",
        "role":"admin"},
      {
        "name":"core-admin-edit",
        "role":"admin"},
      {
        "name":"collection-admin-edit",
        "role":"admin"}
    ],
    "user-role":{
      "solr": "admin"
    }
  }
}
```

Upload public notice configuration set to zookeeper

``` sh
./zkcli.sh -zkhost solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181 -cmd upconfig -confdir ~/downloads/configsets/nm_configs/conf -confname nm_configs
./zkcli.sh -zkhost solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181 -cmd upconfig -confdir ~/downloads/configsets/nmis_configs/conf -confname nmis_configs
./zkcli.sh -zkhost solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181 -cmd putfile /security.json ~/downloads/configsets/security.json
```

Remove old config can be done by

``` sh
./zkcli.sh -cmd clear -z "solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181" /configs/nm_configs
./zkcli.sh -cmd clear -z "solr-1.mycompany.com:2181,solr-2.mycompany.com:2181,solr-3.mycompany.com:2181" /configs/nmis_configs
```

Restart Solr

``` sh
sudo service solr start
```

Create public_notice index collection on **ONLY ONE** node **async id should be unique**

```sh
https://solr-1.mycompany.com:8983/solr/admin/collections?action=CREATE&name=nm&numShards=1&replicationFactor=3&collection.configName=nm_configs&maxShardsPerNode=1&async=1000

https://solr-1.mycompany.com:8983/solr/admin/collections?action=CREATE&name=nmis&numShards=1&replicationFactor=3&collection.configName=nmis_configs&maxShardsPerNode=1&async=1001
```

## Make zookeeper as Service

```sh
sudo vi /etc/init.d/zookeeper
```

Add the following

``` sh
#!/bin/sh
#
# Purpose: This script starts and stops the Zookeeper daemon
#
# chkconfig: - 90 10
# description:  Zookeeper daemon

### BEGIN INIT INFO
# Provides:          zookeeper
# Required-Start:    $network $local_fs $remote_fs
# Required-Stop:     $network $local_fs $remote_fs
# Should-Start:      
# Should-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Controls Zookeeper as a Service
### END INIT INFO

# Where you extracted the Solr distribution bundle
ZK_INSTALL_DIR="/opt/zookeeper"

if [ ! -d "$ZK_INSTALL_DIR" ]; then
  echo "$ZK_INSTALL_DIR not found! Please check the ZK_INSTALL_DIR setting in your $0 script."
  exit 1
fi

# Specify the user to run Zookeeper as; if not set, then Zookeeper will run as root.
# Running Zookeeper as root is not recommended for production environments
RUNAS="solr"

# verify the specified run as user exists
runas_uid="`id -u "$RUNAS"`"
if [ $? -ne 0 ]; then
  echo "User $RUNAS not found! Please create the $RUNAS user before running this script."
  exit 1
fi

case "$1" in
  start|stop|restart|status)
    ZK_CMD="$1"
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit
esac

if [ -n "$RUNAS" ]; then
  su -c "\"$ZK_INSTALL_DIR/bin/zkServer.sh\" $ZK_CMD" - "$RUNAS"
else
  "$ZK_INSTALL_DIR/bin/zkServer.sh" "$ZK_CMD"
fi

exit 0
```

Save the file make it load by etc/init.d

``` sh
sudo chmod 744 /etc/init.d/zookeeper
```

Modify (each node) /etc/init.d/solr add zookeeper

``` sh
# chkconfig: 2345 90
# description: Apache Solr1 Service

### BEGIN INIT INFO
# Required-Start:    $remote_fs $syslog zookeeper
```

Make zookeeper start on boot

```sh
sudo /sbin/chkconfig --add zookeeper
sudo /sbin/chkconfig zookeeper on
```

## Rule-Based Authorization

Create 2 business users for public notice [Reference](https://lucene.apache.org/solr/guide/7_1/basic-authentication-plugin.html)

``` sh
curl --user solr:SolrRocks --noproxy localhost --insecure https://localhost:8983/solr/admin/authentication -H 'Content-type:application/json' -d '{
  "set-user": {"nmis":"toronto",
    "nm":"toronto"}
}'
```

Create role for 2 business collections [Reference](https://lucene.apache.org/solr/guide/7_1/rule-based-authorization-plugin.html)

``` sh
curl --user solr:SolrRocks --noproxy localhost --insecure https://localhost:8983/solr/admin/authorization -H 'Content-type:application/json' -d '{
  "set-permission": {"collection": "nmis","role": "nmis-role"},
  "set-permission": {"collection": "nm","role": ["nmis-role","nm-role"]}
}'
```

Assign proper role to user

``` sh
curl --user solr:SolrRocks --noproxy localhost --insecure https://localhost:8983/solr/admin/authorization -H 'Content-type:application/json' -d '{
  "set-user-role":{"solr": "admin",
    "nmis": "nmis-role",
    "nm": "nm-role"}
}'
```

Download the security.json from zookeeper for reference

``` sh
/opt/solr/server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:2181 -cmd getfile /security.json ./security.json
```

security.json

``` json
{
  "authentication":{
    "blockUnknown":true,
    "class":"solr.BasicAuthPlugin",
    "credentials":{
      "solr":"IV0EHq1OnNrj6gvRCwvFwTrZ1+z1oBbnQdiVC3otuq0= Ndd7LKvVBAaZIF0QAVi1ekCfAJXr1GGfLtRUXhgrF8c=",
      "nmis":"eLU7XWV7ob5aSp3hXefiDCXjdeA7up6IDUbxKaBU2HE= hFOpcS0V/ixMSPO+qI79GRUwG6Sz0AwH0uuXPKmelmM=",
      "nm":"kvoNAU0DYjM5F1ZbSyPq9gpYLJTKUvDzfxf6YDRGwqQ= oGVpM4iRO4AiM/HtiXuag/CmvHHNYX6q/jTJeDBlb58="},
    "":{"v":25}},
  "authorization":{
    "class":"solr.RuleBasedAuthorizationPlugin",
    "permissions":[
      {
        "name":"security-edit",
        "role":"admin",
        "index":1},
      {
        "name":"schema-edit",
        "role":"admin",
        "index":2},
      {
        "name":"config-edit",
        "role":"admin",
        "index":3},
      {
        "name":"core-admin-edit",
        "role":"admin",
        "index":4},
      {
        "name":"collection-admin-edit",
        "role":"admin",
        "index":5},
      {
        "collection":"nmis",
        "role":"nmis-role",
        "index":6},
      {
        "collection":"nm",
        "role":[
          "nmis-role",
          "nmrole"],
        "index":7}],
    "user-role":{
      "solr":[
        "admin",
        "nmis-role",
        "nm-role"],
      "nmis":"nmis-role",
      "nm":"nm-role"},
    "":{"v":24}}}
```

## extra step for Solr 7.1.0 (optional)

```sh
curl --user solr:SolrRocks --noproxy localhost --insecure https://localhost:8983/solr/nmis/config -H 'Content-type:application/json' -d'{
    "set-property" : {"requestDispatcher.requestParsers.enableRemoteStreaming":true},
    "set-property" : {"requestDispatcher.requestParsers.enableStreamBody":true}
}'

curl --user solr:SolrRocks --noproxy localhost --insecure https://localhost:8983/solr/nm/config -H 'Content-type:application/json' -d'{
    "set-property" : {"requestDispatcher.requestParsers.enableRemoteStreaming":true},
    "set-property" : {"requestDispatcher.requestParsers.enableStreamBody":true}
}'
```

## Config Solr Jetty JNDI (optional but recommended)

Get extra library

``` sh
cd /opt/solr/server/lib
wget http://central.maven.org/maven2/org/eclipse/jetty/jetty-plus/9.3.20.v20170531/jetty-plus-9.3.20.v20170531.jar
wget http://central.maven.org/maven2/org/eclipse/jetty/jetty-jndi/9.3.20.v20170531/jetty-jndi-9.3.20.v20170531.jar

cd /opt/solr/server/lib/ext
wget http://central.maven.org/maven2/commons-dbcp/commons-dbcp/1.4/commons-dbcp-1.4.jar
wget http://central.maven.org/maven2/commons-pool/commons-pool/1.6/commons-pool-1.6.jar

cp /opt/share-lib/* /opt/solr/server/lib/ext/
```

Modify contexts/solr-jetty-context.xml

``` sh
vi /opt/solr/server/contexts/solr-jetty-context.xml
```

Add the following JNDI datasource between &lt;Configure&gt; &lt;/Configure&gt;

``` xml
  <New id="nmisDataSource" class="org.eclipse.jetty.plus.jndi.Resource">
    <Arg></Arg>
    <Arg>jdbc/nmisDS</Arg>
    <Arg>
      <New class="org.apache.commons.dbcp.BasicDataSource">
        <Set name="driverClassName">oracle.jdbc.driver.OracleDriver</Set>
        <Set name="url">jdbc:oracle:thin:@//dragon.corp.mycompany.com:1521/NMISDV</Set>
        <Set name="username">nmis_user</Set>
        <Set name="password"><AskDBA></Set>
      </New>
    </Arg>
  </New>
  <New id="nmDataSource" class="org.eclipse.jetty.plus.jndi.Resource">
    <Arg></Arg>
    <Arg>jdbc/nmDS</Arg>
    <Arg>
      <New class="org.apache.commons.dbcp.BasicDataSource">
        <Set name="driverClassName">oracle.jdbc.driver.OracleDriver</Set>
        <Set name="url">jdbc:oracle:thin:@//dragon.corp.mycompany.com:1521/NMISDV</Set>
        <Set name="username">nmis_web</Set>
        <Set name="password"><AskDBA></Set>
      </New>
    </Arg>
  </New>
```