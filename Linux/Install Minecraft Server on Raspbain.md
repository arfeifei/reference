# How to Install Minecraft Server on Raspberry Pi

Contents

*   [Prerequisites](#prerequisites)
*   [Installing Java Runtime Environment](#installing-java-runtime-environment)
*   [Creating Minecraft User](#creating-minecraft-user)
*   [Installing Minecraft on Raspberry Pi](#installing-minecraft-on-raspberry-pi)
    *   [Downloading and Compiling `mcrcon`](#downloading-and-compiling-mcrcon)
    *   [Downloading Minecraft Server](#downloading-minecraft-server)
    *   [Configuring Minecraft Server](#configuring-minecraft-server)
*   [Creating Systemd Unit File](#creating-systemd-unit-file)
*   [Accessing Minecraft Console](#accessing-minecraft-console)
*   [Conclusion](#conclusion)


Raspberry Pi can be used in many different projects. One of the Raspberry Pi’s most popular use case is to turn Raspberry Pi into a game server.

In this tutorial, we will walk you through the process of installing and configuring Minecraft Server on Raspberry Pi 3 or 4.

Minecraft is one of the most popular games of all time. It is a sandbox video game, which allows its players to explore infinite worlds and build everything from simple houses to massive skyscrapers.

[Prerequisites](#prerequisites)
---------------------------------

We’re assuming that you have [Raspbian installed on your Raspberry Pi](https://linuxize.com/post/how-to-install-raspbian-on-raspberry-pi/) . Plex Media Server doesn’t need a graphical interface, so our recommendation is to use the Raspbian Stretch Lite image and [enable SSH](https://linuxize.com/post/how-to-enable-ssh-on-raspberry-pi/) . This way, your Raspberry Pi will have much more available processing power and memory to run the Plex media server.

We’ll use the `mcrcon` utility to connect to the Minecraft server. Install the packages required to build the `mcrcon` tool:
```sh
   $ sudo apt update
```
Enable the GL driver using the `raspi-config` tool:
```sh
   $ sudo raspi-config
```
1.  Navigate to “Advanced Options” using key up or key down and press `Enter`.
2.  Select “GL Driver” and hit `Enter`.
3.  Select “GL (Fake KMS)”, press `Enter`.
4.  Select the “Finish” button, press `Enter`. When prompted “Would you like to reboot now?” select “Yes” and hit `Enter`.

Once the Pi is back online, continue with the next steps.

[Installing Java Runtime Environment](#installing-java-runtime-environment)
-----------------------------------------------------------------------------

Minecraft requires [Java 8](https://linuxize.com/post/install-java-on-raspberry-pi/) or higher to be installed on the system.

We’ll install the headless version of the JRE. This version is more suitable for server applications since it has fewer dependencies and uses less system resources.

To install the headless OpenJRE 8 type:
```sh
   $ sudo apt install openjdk-8-jre-headless
```
Verify the installation by printing the java version:
```sh
   $ java -version

   
    openjdk version "1.8.0_212"
    OpenJDK Runtime Environment (build 1.8.0_212-8u212-b01-1+rpi1-b01)
    OpenJDK Client VM (build 25.212-b01, mixed mode)
```    

[Creating Minecraft User](#creating-minecraft-user)
-----------------------------------------------------

For security purposes, Minecraft should not be run under the root user. We will [create a new system user](https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/) and group with home directory `/opt/minecraft` that will run the Minecraft server:
```sh
   $ sudo useradd -r -m -U -d /opt/minecraft -s /bin/bash minecraft
```
We are not going to set a password for this user. This is good security practice because the user will not be able to login via SSH.

[Installing Minecraft on Raspberry Pi](#installing-minecraft-on-raspberry-pi)
-------------------------------------------------------------------------------

Before starting with the installation process, make sure you [switch to user](https://linuxize.com/post/su-command-in-linux/) “minecraft”:
```sh
   $ sudo su - minecraft
```
Create two directories inside the user home directory:
```sh
   $ mkdir -p ~/{tools,server}
```
*   The `tools` directory will store the `mcrcon` client and the backup script.
*   The `server` directory will contain the actual Minecraft server and its data.

### [Downloading and Compiling `mcrcon`](#downloading-and-compiling-mcrcon)

RCON is a protocol that allows you to connect to the Minecraft servers and execute commands. [mcron](https://github.com/tiiffi/mcrcon) is RCON client built in C.

We’ll download the source code from GitHub and build the `mcrcon` binary.

Navigate to the `~/tools` directory and clone the `Tiiffi/mcrcon` repository from GitHub running the following command:
```sh
   $ cd ~/tools && git clone https://github.com/Tiiffi/mcrcon.git
```
Next, switch to the repository directory:
```sh
   $ cd ~/tools/mcrcon
```
Start the compilation by typing:
```sh
   $ gcc -std=gnu11 -pedantic -Wall -Wextra -O2 -s -o mcrcon mcrcon.c
```
Once completed, you can test it by typing:
```sh
   $ ./mcrcon -h
```
The output will look something like this:
```sh
    Usage: mcrcon [OPTIONS]... [COMMANDS]...
    Sends rcon commands to Minecraft server.
    
    ...
    
    mcrcon 0.6.1 (built: Sep 19 2019 20:52:13)
    Report bugs to tiiffi_at_gmail_dot_com or https://github.com/Tiiffi/mcrcon/issues/
```    

### [Downloading Minecraft Server](#downloading-minecraft-server)

There are several Minecraft server mods such as [Craftbukkit](https://getbukkit.org/download/craftbukkit) or [Spigot](https://www.spigotmc.org/) that allows you to add features (plugins) on your server and further customize and tweak the settings. We will install the latest Mojang’s official vanilla Minecraft server.

Head over to the [Minecraft download page](https://minecraft.net/en-us/download/server/) to get the download link of the latest Minecraft server’s Java archive file (JAR).

At the time of writing, the latest version is `1.16.2`. Before running the next command, you should check the download page for a new version.

Run the following [`wget`](https://linuxize.com/post/wget-command-examples/) command to download the Minecraft jar file in the `~/server` directory 
```sh
   $ wget https://launcher.mojang.com/v1/objects/c5f6fb23c3876461d46ec380421e42b289789530/server.jar -P ~/server
```
### [Configuring Minecraft Server](#configuring-minecraft-server)

Once the download is completed, [navigate](https://linuxize.com/post/linux-cd-command/) to the `~/server` directory and start the Minecraft server:
```sh
   $ cd ~/server
   $ java -Xms512M -Xmx768M -jar server.jar nogui
```
When started for the first time, the server executes some operations and creates the `server.properties` and `eula.txt` files and stops.

    [21:06:23] [main/ERROR]: Failed to load properties from file: server.properties
    [21:06:24] [main/WARN]: Failed to load eula.txt
    [21:06:24] [main/INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
    

To run the server you’ll need to agree to the Minecraft EULA. Open the `eula.txt` file and change `eula=false` to `eula=true`:
```sh
   $ nano ~/server/eula.txt
```
~/server/eula.txt

    #By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula).
    #Thu Sep 19 21:06:24 BST 2019
    eula=true


Close and save the file.

Next, edit the `server.properties` file to enable the rcon protocol and set the rcon password. Open the file using your text editor
```sh
   $ nano ~/server/server.properties
```
Locate the following lines and update their values, as shown below:

~/server/server.properties

    rcon.port=25575
    rcon.password=strong-password
    enable-rcon=true
    

Do not forget to change the `strong-password` to something more secure. If you don’t want to connect to the Minecraft server from remote locations, make sure your firewall blocks the rcon port.

While here, you can also adjust the server’s default properties. For more information about the available settings, check the [server.properties](https://minecraft.gamepedia.com/Server.properties) page.

[Creating Systemd Unit File](#creating-systemd-unit-file)
-----------------------------------------------------------

To run Minecraft as a service, we will create a new Systemd unit file.

Switch back to your sudo user by typing `exit`.

Open your text editor and create a file named `minecraft.service` in the `/etc/systemd/system/`:
```sh
   $ sudo nano /etc/systemd/system/minecraft.service
```
Paste the following configuration:

/etc/systemd/system/minecraft.service

    [Unit]
    Description=Minecraft Server
    After=network.target
    
    [Service]
    User=minecraft
    Nice=1
    KillMode=none
    SuccessExitStatus=0 1
    ProtectHome=true
    ProtectSystem=full
    PrivateDevices=true
    NoNewPrivileges=true
    WorkingDirectory=/opt/minecraft/server
    ExecStart=/usr/bin/java -Xmx768M -Xms512M -jar server.jar nogui
    ExecStop=/opt/minecraft/tools/mcrcon/mcrcon -H 127.0.0.1 -P 25575 -p strong-password stop
    
    [Install]
    WantedBy=multi-user.target
    

Modify the `Xmx` and `Xms` flags according to your Raspberry Pi version and resources. The `Xmx` flag defines the maximum memory allocation pool for a Java virtual machine (JVM), while `Xms` defines the initial memory allocation pool. Also, make sure that you are using the correct `rcon` port and password.

Save and close the file and reload the systemd configuration:
```sh
   $ sudo systemctl daemon-reload
```
Start the Minecraft server by issuing:
```sh
   $ sudo systemctl start minecraft
```
Check the service status with the following command:
```sh
   $ sudo systemctl status minecraft
```
    ● minecraft.service - Minecraft Server
       Loaded: loaded (/etc/systemd/system/minecraft.service; enabled; vendor preset: enabled)
       Active: active (running) since Thu 2019-09-19 21:11:58 BST; 1min 27s ago
     Main PID: 1992 (java)
        Tasks: 17 (limit: 1604)
       Memory: 338.9M
       CGroup: /system.slice/minecraft.service
               └─1992 /usr/bin/java -Xmx768M -Xms512M -jar server.jar nogui
    

The first time you start the service, it will generate several configuration files and directories, including the Minecraft world. Use the `tail` command to monitor the server log file:
```sh
   $ tail -f /opt/minecraft/server/logs/latest.log
```
Once the Minecraft server is started the output will look something like this:

    [21:19:25] [Server-Worker-3/INFO]: Preparing spawn area: 98%
    [21:19:25] [Server thread/INFO]: Time elapsed: 201586 ms
    [21:19:25] [Server thread/INFO]: Done (418.339s)! For help, type "help"
    [21:19:25] [Server thread/INFO]: Starting remote control listener
    [21:19:25] [RCON Listener #1/INFO]: RCON running on 0.0.0.0:25575
    

Enable the Minecraft service to start at boot time automatically:
```sh
   $ sudo systemctl enable minecraft
```
[Accessing Minecraft Console](#accessing-minecraft-console)
-------------------------------------------------------------

To access the Minecraft Console use the `mcrcon` utility. You need to specify the host, rcon port, rcon password and use the `-t` switch which enables the `mcrcon` terminal mode:
```sh
   $ /opt/minecraft/tools/mcrcon/mcrcon -H 127.0.0.1 -P 25575 -p strong-password -t

    Logged in. Type "Q" to quit!
    > 
```

When accessing the Minecraft Console from a remote location, make sure the rcon port is not blocked.

If you are regularly connecting to the Minecraft console, instead of typing this long command, you should create a [bash alias](https://linuxize.com/post/how-to-create-bash-aliases/) .

[Conclusion](#conclusion)
---------------------------

You have successfully installed Minecraft server on your Raspberry Pi. Please note, Minecraft may not run smoothly on systems with low resources.

If you hit a problem or have feedback, leave a comment below.

[java](/tags/java/) [minecraft](/tags/minecraft/) [raspberry pi](/tags/raspberry-pi/)