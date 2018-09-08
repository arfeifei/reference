Things to Do After Minimal RHEL/CentOS 7 Installation

###1.Configure Network with Static IP Address & set hostname
```
#nmcli -d 
#nmtui
#service network restart
#ip addr show
```
###2.Update or Upgrade CentOS Minimal Install
```
#yum update && yum upgrade
```
###3.Install Command Line Web Browser
```
#yum install links
```
###4.Install Apache HTTP Server
```
#yum install httpd
#firewall-cmd --add-service=http
#firewall-cmd --reload
```
###5.Install GCC
```
#yum install gcc
```
###6.Install Java
```
#yum install java
```
###7.Install Nmap lsof to Monitor Open Ports
```
#yum install nmap lsof
#firewall-cmd --list-ports
```
###8. Installing Wget
```
#yum install wget
```
###9. Installing Telnet
```
#yum install telnet
```
###10.Installing Webmin
```
#wget http://prdownloads.sourceforge.net/webadmin/webmin-1.890-1.noarch.rpm
#yum -y install perl perl-Net-SSLeay openssl perl-IO-Tty perl-Encode-Detect
#rpm -U webmin-1.860-1.noarch.rpm
```
###11.Enable Third Party Repositories (epel & nux-dextop)
```
#yum install epel-release
#rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
```
###12.Install 7-zip Utility
```
#yum install p7zip
#7za x xxx.zip
```
###13.Install Rootkit Hunter
```
#yum install rkhunterum
#rkhunter --check
```
###14.Install Docker-CE
* Uninstall old versions
```
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
                  
```
* Install using the repository
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

```
* INSTALL DOCKER CE
```
$ sudo yum install docker-ce
# make current user can user docker
$ sudo usermod -a -G docker $USER

# make it boot automatically 
$ sudo systemctl enable docker
$ sudo systemctl start docker
```
###15.Install Docker Compose on Linux systems
```
$ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
```
###16.Install NodeJs
```
# Login as root
# yum install -y gcc-c++ make
# curl -sL https://rpm.nodesource.com/setup_10.x | bash -
# yum install -y nodejs
```
###17.Install Gnome-Desktop
```
# yum group install "GNOME Desktop" "Graphical Administration Tools"
# ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target
```
###17.Install XRdp
```
# yum -y install xrdp
# service xrdp start
# service xrdp-sesman start
# systemctl enable xrdp.service
# firewall-cmd --permanent --zone=public --add-port=3389/tcp
# firewall-cmd --reload
```

[Reference](https://www.tecmint.com/things-to-do-after-minimal-rhel-centos-7-installation)