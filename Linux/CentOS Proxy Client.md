Proxy Client : CentOS

# Configure proxy settings like follows on CentOS Client.
## for enviroment
``` sh
[root@client ~]# vi /etc/profile
# add follows to the end (set proxy settings to the environment variables)
MY_PROXY_URL="http://prox.srv.world:3128/"
HTTP_PROXY=$MY_PROXY_URL
HTTPS_PROXY=$MY_PROXY_URL
FTP_PROXY=$MY_PROXY_URL
http_proxy=$MY_PROXY_URL
https_proxy=$MY_PROXY_URL
ftp_proxy=$MY_PROXY_URL
export HTTP_PROXY HTTPS_PROXY FTP_PROXY http_proxy https_proxy ftp_proxy
[root@client ~]# source /etc/profile
# it's OK all, but it's possible to set proxy settings for each application like follows
```
## for yum
``` sh
[root@client ~]# vi /etc/yum.conf
# add to the end
proxy=http://prox.srv.world:3128/
```
## for wget
``` sh
[root@client ~]# vi /etc/wgetrc
# add to the end
http_proxy = http://prox.srv.world:3128/
https_proxy = http://prox.srv.world:3128/
ftp_proxy = http://prox.srv.world:3128/
```