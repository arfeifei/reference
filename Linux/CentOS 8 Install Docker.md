CentOS 8 install Docker
------------------------------------------------


Install the containerd.io package for Docker

```sh
sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
```
The benefit of installing containerd.io manually, is that you can now install the latest version of docker-ce, as opposed to installing an older, specific version. Install the latest docker-ce release with the command:

```sh
sudo dnf install docker-ce
```
Once that installs, you should now be able to upgrade CentOS without issue. 
