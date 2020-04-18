# Docker Proxy Setting

Tested on CentOS 8

From [https://docs.docker.com/network/proxy/](https://docs.docker.com/network/proxy/)

## 1. Docker Client Proxy Setting:

### config.json
> On the Docker client, create or edit the file **~/.docker/config.json**
```json
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://proxyserver:3128",
     "httpsProxy": "http://proxyserver:3128",
     "noProxy": "*.test.example.com,.example2.com"
   }
 }
}
```
### Enviroment variables
```sh
|   Variable   |           Dockerfile example               |            docker run Example              |
|--------------|:------------------------------------------:|-------------------------------------------:|
| HTTP_PROXY   |  ENV HTTP_PROXY "http://127.0.0.1:3001"    | --env HTTP_PROXY="http://127.0.0.1:3001"   |
| HTTPS_PROXY  |  ENV HTTPS_PROXY "http://127.0.0.1:3001"   | --env HTTPS_PROXY="http://127.0.0.1:3001"  |
| NO_PROXY     |  ENV NO_PROXY "*.test.example.com"       	| --env NO_PROXY="*.test.example.com "       |

```
## 2. Docker Daemon Proxy Setting
https://docs.docker.com/config/daemon/systemd/

The Docker daemon uses the HTTP_PROXY, HTTPS_PROXY, and NO_PROXY environmental variables in its start-up environment to configure HTTP or HTTPS proxy behavior. You **cannot** configure these environment variables using the **daemon.json** file.

1.Create a systemd drop-in directory for the docker service:
```sh
$ sudo mkdir -p /etc/systemd/system/docker.service.d
```
2.Create a file called */etc/systemd/system/docker.service.d/http-proxy.conf* that adds the **HTTP_PROXY** environment variable:
```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128/" "NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
```
Or, if you are behind an HTTPS proxy server, create a file called /etc/systemd/system/docker.service.d/https-proxy.conf that adds the HTTPS_PROXY environment variable:
```ini
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:443/" "NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
```
## 3. Trouble shooting tips

### 1. Error response from daemon: Get https://registry-1.docker.io/v2/

docker daemon behind proxy need set http-proxy.conf or https-proxy.conf as mentioned above.

### 2. No route to host

When docker cannot access host network or access internet. Got error messages like  **no route to host** when run
```sh
docker run busybox nslookup google.com
```
config firewallD to allow docker containers free access to the host's network or internet
```sh
sudo firewall-cmd --permanent --zone=trusted --change-interface=docker0
sudo firewall-cmd --reload
```
