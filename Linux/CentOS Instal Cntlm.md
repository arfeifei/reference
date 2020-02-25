1) Download cntlm rpm package from http://sourceforge.net/projects/cntlm/files/cntlm/

2) Login as root

3) Run command:
    $ rpm -ivh cntlm-*.rpm

4a) Obtain password hash for the configuration file in step 4b (do not put plaintext password in configuration)
    $ cntlm -H -d <domain> -u <username>

4b) Configure CNTLM:
    $ vi /etc/cntlm.conf
    
5) Export proxy settings:
    $ vi ~/.bashrc
    vi /etc/profile.d/proxy.sh
    vi /etc/profile
    export http_proxy=http://localhost:3128
    export https_proxy=${http_proxy}
    export ftp_proxy=${http_proxy}
    vi /etc/dnf/dnf.conf
    
6) Run command:
    $ . ~/.bashrc
 
7) Create directory
   $ mkdir /var/run/cntlm
   $ chgrp cntlm /var/run/cntlm/
   $ chmod g+w /var/run/cntlm/

8) Enable CNTLM to start automatically:
    $ chkconfig cntlmd on  (systemctl enable cntlmd)
    $ service cntlmd start (systemctl start cntlmd)
