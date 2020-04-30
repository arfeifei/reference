Enable snaps on CentOS and install janus-gateway
------------------------------------------------

Snaps are applications packaged with all their dependencies to run on all popular Linux distributions from a single build. They update automatically and roll back gracefully.

Snaps are discoverable and installable from the [Snap Store](/store), an app store with an audience of millions.

![](https://res.cloudinary.com/canonical/image/fetch/f_auto,q_auto,fl_sanitize,w_169,h_159/https://assets.ubuntu.com/v1/acf876d9-Distro_Logo_CentOS.svg)

### Enable snapd

Snap is available for CentOS 7.6+, and Red Hat Enterprise Linux 7.6+, from the [Extra Packages for Enterprise Linux](https://fedoraproject.org/wiki/EPEL) (EPEL) repository. The EPEL repository can be added to your system with the following command:

```sh
sudo yum install epel-release
```
Snap can now be installed as follows:

```sh
sudo yum install snapd
```
Once installed, the _systemd_ unit that manages the main snap communication socket needs to be enabled:

```sh
sudo systemctl enable --now snapd.socket
```
To enable _classic_ snap support, enter the following to create a symbolic link between `/var/lib/snapd/snap` and `/snap`:

```sh
sudo ln -s /var/lib/snapd/snap /snap
```

Either log out and back in again, or restart your system, to ensure snap’s paths are updated correctly.

### Install janus-gateway

To install janus-gateway, simply use the following command:

```sh
sudo snap install janus-gateway
```