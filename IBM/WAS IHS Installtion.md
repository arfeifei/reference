# WAS IHS installation

1. Install WAS9ND
    * Start Program [None]

2. Install IBM HTTP SERVER
    * IBM HTTP SERVER port [80]

3. Install WebServer Plug-in for IBM HTTP Server

4. Install WebSphere Customization Toolbox
    * Start Program [None]

5. Create One Deployment Manager profile and Custom profile.
    1. <was_rootdirectory>/bin/ProfileManagement/pmt.sh
    2. Create default certificate for dmgr
       * default key-store
       * password: WebAS
       * Port 9060
       * SOAP port 8879
    3. Launch the First steps console or run command line
        1. start dmgr
            * <dmgr_root>/bin/startServer.sh dmgr
        2. check status
            * <dmgr_root>/bin/serverStatus.sh -all
    4. create Custom profile > Advanced profile creation
        1. Federate node at creation time
            * Deployment manager SOAP port 8879
        2. Federate node later
            * <node_root>/bin/addNode.sh localhost 8879
        3. Create default certificate for custom01 default key-store password:WebAS
    5. Uncheck Launch the First steps console

6. Create Application Server in Deployment Manager Admin Console.
    1. http://host:9060/ibm/console/login.do
    2. Servers > Server Types > WebSphere application servers> [new]
        * server1

7. Configure IBM HTTP SERVER with WAS using Websphere Customization Toolbox.
    1. <toolbox>/WCT/wct.sh
    2. Web Server Plug-ins runtime locations [Add]
        * name: http
        * location: /opt/IBM/WebSphere/Plugins
    3. Web Server Plug-in Configurations [Create]
        * IBM HTTP Server > 64 bit
        * existing httpd.conf file: /opt/IBM/HTTPServer/conf/httpd.conf
        * Web server port: 80
        * HTTP Administration Port: 8008
        * HTTP Administration Authentication: UserID: wasadmin Password:toronto
        * UserID: puser Group: puser
        * webserver1
        * Local Install location off WebSphere Application Server: /opt/IBM/WebSphere/AppServer
        * profile: Dmgr01
        * click [configure]
        * Finish

8. Create unmanaged node(IBM HTTP SERVER) on DMGR Admin Console.
    1. http://host:9060/ibm/console/login.do
    2. System administration > Nodes > Add Node > Unmanaged node 
        * Name: IhsNode
        * HostName: localhost
    3. Servers > Server Types > Web servers> [new]
        * Node: IhsNode
        * Server name: webserver1
        * fill with step 7.3 information

9. Done

## Linux ONLY run IBM HTTP Server as non-Root user

    1. As root, stop IHS:
       /opt/IBM/HTTPServer/bin/apachectl stop

    2. Run setcap against the httpd process:
       $ setcap CAP_NET_BIND_SERVICE+ep /opt/IBM/HTTPServer/bin/httpd

    3. Change ownership on various files:
       chown puser /opt/IBM/HTTPServer/logs
       chown puser /opt/IBM/HTTPServer/logs/*
       chown puser /opt/IBM/WebSphere/Plugins/config/webserver1/plugin-key.*
       chown puser /opt/IBM/WebSphere/Plugins/logs/webserver1
       chown puser /opt/IBM/WebSphere/Plugins/logs/webserver1/*

    4. Update load library configuration:
       cd /etc/ld.so.conf.d/
       echo /opt/IBM/HTTPServer/lib > httpd-lib.conf
       echo /opt/IBM/HTTPServer/gsk8/lib64 >> httpd-lib.conf
       mv /etc/ld.so.cache /etc/ld.so.cache.old
       /sbin/ldconfig

    5. Start IHS as puser
       sudo -u puser /opt/IBM/HTTPServer/bin/apachectl start