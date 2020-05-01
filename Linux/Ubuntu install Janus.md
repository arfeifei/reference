Ubuntu 下 Janus Server 搭建笔记
=========================

1 简介
====

[Janus](https://github.com/meetecho/) 是一个开源的，通过 C 语言实现了对 WebRTC 支持的 Gateway；Janus 自身实现得很简单，提供插件机制来支持不同的业务逻辑，配合官方自带插件就可以用来实现高效的 Media Server 服务。  
本文主要介绍如何在 Centos 7.0 下搭建起 janus 服务器，实现 janus 官方 Demo 浏览器与 Android APP Demo（janus-gateway-android）之间的音视频通话。

目前浏览器仅支持 FireFox 浏览器，因为 Chrome 浏览器打开音视频采集的话需要 HTTPS 加密访问！

Janus 官网：[https://janus.conf.meetecho.com/index.html](https://janus.conf.meetecho.com/index.html])

参考文档：[https://github.com/meetecho/janus-gateway](https://github.com/meetecho/janus-gateway)

2 下载编译 Janus
============

编译运行 Janus Server 需要依赖较多的一些第三方库

2.1 安装依赖
--------

Ubuntu 下通过 aptitude 批量安装依赖工具包，这里建议 Ubuntu 镜像源（/etc/apt/source.list）不要为了追求速度而改用了国内的某些镜像源，如 网易 163，这可能会导致某些工具包下载失败，建议依然使用官方自带的镜像源。  
yum list | grep nice  
yum install gtk-doc  
批量安装命令：
```sh
    #!/bin/sh
    yum install -y epel-release && \
    yum update -y && \
    yum install -y deltarpm && \
    yum install -y openssh-server which file curl zip unzip wget && \
    yum install -y libmicrohttpd-devel jansson-devel libnice-devel glib22-devel opus-devel libogg-devel pkgconfig  gengetopt libtool autoconf automake make gcc gcc-c++ git cmake libconfig-devel openssl-devel
    
    yum install -y autoconf automake libtool
    
    #upgrade libsrtp 1.5.4
    wget https://github.com/cisco/libsrtp/archive/v1.5.4.tar.gz
    tar xfv v1.5.4.tar.gz
    cd libsrtp-1.5.4
    ./configure --prefix=/usr --enable-openssl --libdir=/usr/lib64
    make shared_library && make install
    
    cd -
    
    wget https://github.com/cisco/libsrtp/archive/v2.0.0.tar.gz
    tar xfv v2.0.0.tar.gz
    cd libsrtp-2.0.0
    ./configure --prefix=/usr --enable-openssl
    make shared_library && make install
    
    cd -
    
    #install sofia-sip for sip-gateway plugin
    wget https://sourceforge.net/projects/sofia-sip/files/sofia-sip/1.12.11/sofia-sip-1.12.11.tar.gz
    tar zxf sofia-sip-1.12.11.tar.gz && cd sofia-sip-1.12.11 && ./configure --prefix=/usr CFLAGS=-fno-aggressive-loop-optimizations && make && make install
    
    cd -
    
    #install usrsctp for Data channel support
    git clone https://github.com/sctplab/usrsctp && cd usrsctp && \
    ./bootstrap && \
    ./configure --prefix=/usr && make && make install
    
    cd -
    
    #install libwebsocket for android or ios instead of http/https
    git clone https://github.com/warmcat/libwebsockets && \
    mkdir libwebsockets/build && cd libwebsockets/build && \
    cmake -DMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_C_FLAGS="-fpic" .. && \
    make && make install
    
    cd -
    
    #Janus
    #if cannot configure plugin sofia,Perhaps you should add the directory containing `sofia-sip-ua.pc' to the PKG_CONFIG_PATH environment variable,
    #for example centos7 :export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
    export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
    #if cannot load libsofia-sip-ua.so.0 , try ldconfig -v
    git clone https://github.com/meetecho/janus-gateway.git && \
    cd janus-gateway &&\
    sh autogen.sh && \
    ./configure --prefix=/opt/janus --disable-rabbitmq --disable-docs --disable-libsrtp2  &&\
    make && make install && make configs
    
```

实际上，在新版本的janus，libsrtp1.5版本是编译不过的。  
如果出现某个工具包下载失败，请修改镜像源为官方地址，并执行以下命令

    yum -y update 
    yum -y upgrade 
    

以更新镜像源，完成后重新安装。

2、 安装 WebSocket
---------------

janus 支持 WebSocket 是可选项，如果不安装，编译 janus 时，默认不支持 WebSocket 的链接请求，而 Android APP Demo 是通过 WebSocket 与 janus 进行通信的，因为我们希望 Android APP Demo 能与浏览器（HTTP）进行视频通话，所以就必须要在编译 janus 时支持 WebSocket。

依次执行以下命令，分别进行下载，编译，安装：
```sh
    git clone https://github.com/warmcat/libwebsockets.git
    cd libwebsockets
    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_C_FLAGS="-fpic" ..
    make && make install
    
```

安装成功后，在编译 janus 时，janus 默认会增加对 WebSocket 的集成，或者通过增加编译参数 --enable-websockets 打开 WebSocket 开关，或 --disable-websockets 关闭 WebSocket 开关。

3、 安装 Http Server
-----------------

Janus 源码目录下的 html 下自带 Web Demo（html & JavaScript ），Janus 编译完成并 Start 以后，需要通过 http server 访问 Janus Web Demo，其中包括：
```sh
        Echo Test:             yes
        Streaming:             yes
        Video Call:            yes
        SIP Gateway:           yes
        Audio Bridge:          yes
        Video Room:            yes
        Voice Mail:            yes
        Record&Play:           yes
        Text Room:             yes
```

以上 janus 插件均可通过相应的 http 链接进行访问体验。

​
以下介绍一种快速，便捷，轻巧的 HTTP Server 安装方式：

通过 Node.js (基于 Chrome V8 引擎的 JavaScript 运行环境) 进行安装，首先安装 Node.js：
```sh
    yum install -y nodejs
``` 

安装成功后，通过 npm (npm 是 Node.js 的包管理器，是全球最大的开源库生态系统) 进行安装 httpserver：
```sh
    npm -g install http-server
```

启动方式：

进入到 html 目录，执行 http-server 命令即可，如：
```sh
    gobert@gobert-ThinkPad-X230:~/OpenSource/janus-gateway$ cd html/
    gobert@gobert-ThinkPad-X230:~/OpenSource/janus-gateway/html$ http-server 
    Starting up http-server, serving ./
    Available on:
      http://127.0.0.1:8080
      http://100.100.32.64:8080
    Hit CTRL-C to stop the server
```

输入 http url 即可访问。

**注：需首先 build & start janus Server！**

4、 安装 libsrtp
-------------

Janus 需要至少 version 1.5 以上的 libsrtp，如果系统中已经安装了 libsrtp，则首先卸载后，手动安装新版本，这里我们安装 libsrtp 2.0，依次执行以下命令：
```sh
    wget https://github.com/cisco/libsrtp/archive/v2.0.0.tar.gz
    tar xfv v2.0.0.tar.gz
    cd libsrtp-2.0.0
    ./configure --prefix=/usr --enable-openssl
    make shared_library && make install
```

5、 编译 Janus
-----------

通过 Git 下载 Janus 源码，并编译安装：
```sh
    git clone https://github.com/meetecho/janus-gateway.git
    sh autogen.sh
    ./configure --prefix=/opt/janus --enable-websockets --enable-docs
    make
    make install
```

configure 执行成功后，会输出 janus 所支持的 协议及插件，如下：
```sh
    libsrtp version:           2.0.x
    SSL/crypto library:        OpenSSL
    DTLS set-timeout:          not available
    DataChannels support:      no
    Recordings post-processor: no
    TURN REST API client:      yes
    Doxygen documentation:     no
    Transports:
        REST (HTTP/HTTPS):     yes                //支持 HTTP 访问
        WebSockets:            yes (new API)      //支持 WebSocket 访问
        RabbitMQ:              no
        MQTT:                  no
        Unix Sockets:          yes
    Plugins:
        Echo Test:             yes
        Streaming:             yes
        Video Call:            yes
        SIP Gateway:           yes
        Audio Bridge:          yes
        Video Room:            yes
        Voice Mail:            yes
        Record&Play:           yes
        Text Room:             yes
    Event handlers:
        Sample event handler:  yes
```

6、 运行 Janus
-----------

如果全部安装以上步骤进行编译的 janus ，那么 janus 的全局配置文件存放目录为 ：

```
    /opt/janus/etc/janus/janus.cfg
```

或者在启动 janus 时，加上相应的启动参数，参数可通过 janus --help 查看；

janus 默认的配置中是没有 WebSocket 的配置的，直接启动 Janus 会因没有 WebSocket 配置文件而报错。幸运的是在配置目录中 Janus 已经给我们提供了一个 WebSocket 的示例配置文件 ： janus.transport.websockets.cfg.sample，（如果我们要通过 WebSocket 连接 Janus，则需要有个 WebSocket 的配置文件）这里我们可以直接拷贝这个示例文件：
```sh
    cp janus.transport.websockets.cfg.sample janus.transport.websockets.cfg
```


通过查看此配置文件，可以得知 Janus 默认的 WebSocket 的端口号为 8188，**记住这个端口号，在 Android APP Demo 中会使用到！**

启动 Janus：
```
    /opt/janus/bin/janus --debug-level=7 --log-file=$HOME/janus-log
```
根据需要可以选择是否加上后面两个启动参数。

三、 视频通话联调测试
===========

我们使用 PC 下的 浏览器 与 Android APP Demo 进行联调。

1、 启动 Web Demo
--------------

进入到 janus 目录下的 html 目录，启动 http-server
```
    gobert@gobert-ThinkPad-X230:~$ cd ~/OpenSource/janus-gateway/html/
    gobert@gobert-ThinkPad-X230:~/OpenSource/janus-gateway/html$ http-server 
    Starting up http-server, serving ./
    Available on:
      http://127.0.0.1:8080
      http://192.168.1.15:8080
    Hit CTRL-C to stop the server
```

这样外部便可以通过 http://192.168.1.15:8080 进行访问了，进入首页后，找到 videoRoom，Start

2、 启动 Android APP Demo
----------------------

*   下载:
```
    git clone git@github.com:Computician/janus-gateway-android.git
```

*   修改源代码
    
    janus-gateway-android 支持两个 Demo 测试：EchoTest 和 VideoRoom，默认情况下会启用 EchoTest，这个 Demo 仅仅是连接服务器后，将数据再发回本地进行本地测试，我们要改为与房间内的其它用户（浏览器）进行视频通话，则需要启用另外一个测试用例 VideoRoom，按照如下方式修改代码：
    
    JanusActivity.java 类中新增 VideoRenderer.Callbacks 数组（视频房间中可能会有多人）,暂定义为 2 个，实际连接人数不要超过此数字：
```java
        private VideoRenderer.Callbacks remoteRenders[] = new VideoRenderer.Callbacks[3];
```
    
    OnCreate 方法中初始化以上定义的数组：
```java
        remoteRenders[0] = VideoRendererGui.create(0, 0, 25, 25, VideoRendererGui.ScalingType.SCALE_ASPECT_FILL, true);
``` 
    
    APP Demo 是通过 WebSocket 连接 Janus Server，所以修改 VideoRoomTest.java 中 JANUS\_URL 地址为我们启动的 Janus 服务器 WebSocket 地址，IP 为 janus server 地址，端口默认为 8188：
```java
        private final String JANUS_URI = "ws://100.100.32.64:8188";
```    
    ​
    
*   编译安装
    
    通过 Android studio 进行编译安装到 Android 机。
    

3、联调测试
------

Janus Server 默认会开启两个视频房间：1234 和 5678，分别使用 VP8 和 VP9 视频编码器，所以我们通过 Brower 和 Android APP Demo 进行联调测试时，暂不需要设置房间 ID。


修改/opt/janus/etc/janus/janus.jcfg

```ini
nat: {  
stun_server = "stun.voip.eutelia.it"  
stun_port = 3478  
nice_debug = false
```
