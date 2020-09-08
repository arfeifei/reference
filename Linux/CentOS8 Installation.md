[CentOS](https://linuxhint.com/category/centos/)

Installing CentOS 8 using NetBoot ISO Image
===========================================

11 months ago

by [Shahriar Shovon](https://linuxhint.com/author/shahriar_shovon/)

CentOS 8 DVD ISO installation image is very large in size. If you have limited internet connection, or a low capacity (< 8 GB) USB thumb drive, then CentOS 8 DVD ISO installation image is not a good choice for you. A better solution for you in that case is using the CentOS 8 NetBoot/NetInstall ISO installation image. It’s about 500-600 MB in size and will fit in even as small as 1 GB USB thumb drive.  In this article, I am going to show you how to install CentOS 8 using the CentOS 8 NetBoot ISO installation image. So, let’s get started.  

    

Downloading CentOS 8 NetBoot/NetInstall ISO Installation Image:
===============================================================

First, visit the [CentOS 8 official ISO mirror page](http://isoredirect.centos.org/centos/8/isos/x86_64/).

Once the page loads, click on a mirror link that is geographically closer to you.

[![](https://linuxhint.com/wp-content/uploads/2019/10/1-21.png)](https://linuxhint.com/wp-content/uploads/2019/10/1-21.png)

Now, click on the **CentOS-8-x86\_64-1905-boot.iso** (about **534 MB**) file.

[![](https://linuxhint.com/wp-content/uploads/2019/10/2-21.png)](https://linuxhint.com/wp-content/uploads/2019/10/2-21.png)

Your browser should start downloading the CentOS 8 NetBoot ISO installation image. It may take a while to complete.

[![](https://linuxhint.com/wp-content/uploads/2019/10/3-20.png)](https://linuxhint.com/wp-content/uploads/2019/10/3-20.png)

Making a Bootable USB Thumb Drive of CentOS 8 NetBoot ISO Image:
----------------------------------------------------------------

You can use Rufus, Etcher, UNetbootin, Linux dd command and many more tools to create a bootable USB thumb drive of CentOS 8 NetBoot ISO image. In this article, I am going to use Rufus.

First, visit the [official website of Rufus](https://rufus.ie/). Then, click on Rufus Portable link.

[![](https://linuxhint.com/wp-content/uploads/2019/10/4-20.png)](https://linuxhint.com/wp-content/uploads/2019/10/4-20.png)

Your browser should download Rufus portable.

[![](https://linuxhint.com/wp-content/uploads/2019/10/5-19.png)](https://linuxhint.com/wp-content/uploads/2019/10/5-19.png)

Once Rufus is downloaded and CentOS 8 NetBoot ISO installation image is downloaded, insert a USB thumb drive and open Rufus. Then, click on **SELECT**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/6-19.png)](https://linuxhint.com/wp-content/uploads/2019/10/6-19.png)

Now, select the CentOS 8 NetBoot ISO installation image using the File Picker and click on **Open**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/7-19.png)](https://linuxhint.com/wp-content/uploads/2019/10/7-19.png)

Now, click on **START**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/8-19.png)](https://linuxhint.com/wp-content/uploads/2019/10/8-19.png)

Click on **OK**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/9-20.png)](https://linuxhint.com/wp-content/uploads/2019/10/9-20.png)

Click on **OK**.

**NOTE:** If you have any important data on the USB thumb drive, move them somewhere safe before you click on **OK**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/10-19.png)](https://linuxhint.com/wp-content/uploads/2019/10/10-19.png)

Rufus is copying the contents of the ISO installation image to the USB thumb drive. It may take a while to complete.

[![](https://linuxhint.com/wp-content/uploads/2019/10/11-19.png)](https://linuxhint.com/wp-content/uploads/2019/10/11-19.png)

Once the USB thumb drive is **READY**, click on **START**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/12-18.png)](https://linuxhint.com/wp-content/uploads/2019/10/12-18.png)

### Installing CentOS 8 from the NetBoot Image:

Now, insert the bootable USB thumb drive in your computer and boot from it.

Once you see the following GRUB menu, select **Install CentOS Linux 8.0.1905** and press **<Enter>**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/13-18-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/13-18.png)

Once the CentOS 8 GUI installer starts, select your language and click on **Continue**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/14-17-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/14-17.png)

Now, click on **Network & Host Name**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/15-18-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/15-18.png)

Now, type in a host name and click on **Apply**. Then, click on the toggle button on the top right corner to enable the network adapter.

[![](https://linuxhint.com/wp-content/uploads/2019/10/16-17-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/16-17.png)

If your network is configured with DHCP, the network adapter should get an IP address from your Router.

[![](https://linuxhint.com/wp-content/uploads/2019/10/17-15-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/17-15.png)

If you want to manually configure the network, then click on **Configure**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/18-15-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/18-15.png)

Here, you have some connectivity options in the **General** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/19-16-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/19-16.png)

You can configure Ethernet protocol properties from the **Ethernet** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/20-14-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/20-14.png)

If your network provider requires authentication, you can configure it from the **802.1X Security** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/21-14-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/21-14.png)

You can configure Data Center Bridging (DCB) from the **DCB** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/22-14-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/22-14.png)

You can configure network proxy from the **Proxy** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/23-13-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/23-13.png)

You can configure IPv4 IP settings from the **IPv4 Settings** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/24-12-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/24-12.png)

You can also configure IPv6 IP settings from the **IPv6 Settings** tab.

[![](https://linuxhint.com/wp-content/uploads/2019/10/25-12-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/25-12.png)

Once you’re done with the network configuration, click on **Done**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/26-12-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/26-12.png)

To configure a software repository, click on **Installation Source**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/27-11-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/27-11.png)

By default, **Closest mirror** is selected. It should find a CentOS 8 mirror automatically.

[![](https://linuxhint.com/wp-content/uploads/2019/10/28-12-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/28-12.png)

If you want to use a specific HTTP/HTTPS or FTP or NFS installation source, then you can select it from **On the network** dropdown menu.

[![](https://linuxhint.com/wp-content/uploads/2019/10/29-13-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/29-13.png)

Then, select the **URL type** from the dropdown menu.

[![](https://linuxhint.com/wp-content/uploads/2019/10/30-12-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/30-12.png)

I am going to use the official CentOS 8 repository using the HTTP repository URL [http://mirror.centos.org/centos/8/BaseOS/x86\_64/os/](http://mirror.centos.org/centos/8/BaseOS/x86_64/os/)

[![](https://linuxhint.com/wp-content/uploads/2019/10/31-13-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/31-13.png)

You can also setup a proxy for the installation source repository. To do that, click on **Proxy setup…**

[![](https://linuxhint.com/wp-content/uploads/2019/10/32-12-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/32-12.png)

Now, to configure proxy, check **Enable HTTP Proxy**, type in your proxy configuration and click on **OK**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/33-9-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/33-9.png)

If you want to enable additional custom repositories, click on the **+** button.

[![](https://linuxhint.com/wp-content/uploads/2019/10/34-9-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/34-9.png)

Now, type in your required repository information. The repository should be added.

You can also use repository specific proxy from here if you want.

[![](https://linuxhint.com/wp-content/uploads/2019/10/35-8-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/35-8.png)

Once you’re done, click on **Done**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/36-8-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/36-8.png)

The installation source is being configured as you can see in the screenshot below.

[![](https://linuxhint.com/wp-content/uploads/2019/10/37-7-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/37-7.png)

Once the installation source is configured, click on **Installation Destination**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/38-7-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/38-7.png)

Now, select a hard drive and partition it.

I am installing CentOS 8 using the NetBoot ISO image on a virtual machine. So, I am going to select **Automatic** partitioning. If you want to do manual partitioning, then check my article [How to Install CentOS 8 Server](https://linuxhint.com/install_centos_8_server/).

Once you’re done with the hard disk partitioning, click on **Done**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/39-6-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/39-6.png)

Now, click on **Software Selection**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/40-7-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/40-7.png)

If you want to install CentOS 8 Server with graphical user interface (GNOME), then select **Server with GUI** environment.

If you want to install CentOS 8 headless server (without graphical user interface), then select **Server** or **Minimal Install** environment.

If you want to use CentOS 8 on your desktop or laptop, then select **Workstation** environment.

If you want to configure CentOS 8 for running KVM/QEMU virtual machines, then select **Virtualization Host** environment.

Once you’ve selected a suitable environment, click on **Done**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/41-7-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/41-7.png)

Now, to set up a time zone, click on **Time & Date**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/42-6-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/42-6.png)

Now, select your **Region** and **City** and click on **Done**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/43-6-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/43-6.png)

Now, click on **Begin Installation**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/44-4-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/44-4.png)

As you can see, the CentOS 8 installer is downloading all the required packages from the internet.

[![](https://linuxhint.com/wp-content/uploads/2019/10/45-4-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/45-4.png)

Now, you have to create a user account. To do that, click on **User Creation**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/46-2-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/46-2.png)

Now, type in all your personal information, check **Make this user administrator** and click on **Done**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/47-2-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/47-2.png)

The installation should continue.

[![](https://linuxhint.com/wp-content/uploads/2019/10/48-1-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/48-1.png)

Once the packages are downloaded, they will be installed one by one.

[![](https://linuxhint.com/wp-content/uploads/2019/10/49-1-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/49-1.png)

Once the installation is complete, click on **Reboot**.

[![](https://linuxhint.com/wp-content/uploads/2019/10/50-1-1024x768.png)](https://linuxhint.com/wp-content/uploads/2019/10/50-1.png)

From the next time, CentOS 8 should boot from the hard drive. Now, you should be able to use the username and password that you’ve set during the installation to login.

[![](https://linuxhint.com/wp-content/uploads/2019/10/51-1-1024x145.png)](https://linuxhint.com/wp-content/uploads/2019/10/51-1.png)

As you can see, I am running **CentOS 8** and the Linux kernel version is **4.18.0**.

$ uname \-r  
$ cat /etc/redhat-release

[![](https://linuxhint.com/wp-content/uploads/2019/10/52-2.png)](https://linuxhint.com/wp-content/uploads/2019/10/52-2.png)

So, that’s how you install CentOS 8 using the NetBoot ISO installation image.