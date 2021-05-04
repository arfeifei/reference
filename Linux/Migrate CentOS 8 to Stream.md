# Migrate to CentOS Stream. This can be achieved in the following simple steps:

```sh
$ sudo  dnf install centos-release-stream
$ sudo  dnf swap centos-{linux,stream}-repos
$ sudo  dnf distro-sync
```

