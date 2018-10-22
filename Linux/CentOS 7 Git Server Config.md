# Configure Git Server on CentOS 7

## Server

```ssh
$ yum install git-core
$ sudo useradd git
$ sudo passwd git
```

## Client (Local Machine)

```ssh
$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub | ssh git@remote-server "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```

## Server

```ssh
$ sudo su git
$ mkdir -p /home/git/project-1.git
$ cd /home/git/project-1.git
$ git init --bare
Initialized empty Git repository in /home/git/project-1.git
```

## Client (Local Machine)

```ssh
$ mkdir -p /home/user/dev/project
$ cd /home/user/dev/project
$ git init
Initialized empty Git repository in /home/user/dev/project
$ git add . // Normal work with git
$ git commit -m "blah blah"
$ git remote add origin git@remote-server:project-1.git
$ git push origin master
```

if git server asks for password then review /var/log/secure in server

## Server
```ssh
$ tail -f /var/log/secure
```
and search for
```
Authentication refused: bad ownership or modes for directory /home/git/.ssh
or
Authentication refused: bad ownership or modes for file /home/git/.ssh/authorized_keys
```

To solve this problem we need to change mode and permission of .ssh folder and authorized_keys file

## Server
```ssh
$ sudo su git
$ chmod 600 /home/git/.ssh
$ chmod 700 /home/git/.ssh/authorized_keys
$ systemctl restart sshd.service // restart ssh service
```

and push again

## Client

```ssh
$ git push origin master
```

## Usage
after that go to the server machine and clone the repo on client machine :
```ssh
git clone ssh://root@192.168.1.71/home/user/dev/project
```

[here 192.168.1.71 is IP of client machine and /home/user/dev/project is the path of repo on client machine