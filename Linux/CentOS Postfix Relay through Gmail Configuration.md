Configure a Postfix Relay through Gmail on CentOS 7

## Requirements
* CentOS 7 or Red Hat Enterprise Linux 7
* Valid Gmail or Google App credentials
## Install Packages
Make sure **Postfix**, the SASL authentication framework, and **mailx** are all installed.
``` sh
yum -y install postfix cyrus-sasl-plain mailx
```
Postfix will need to be restarted before the SASL framework will be detected.
``` sh
systemctl restart postfix
```
Postfix should also be set to start on boot.
``` sh
systemctl enable postfix
```
## Configure Postfix
Open the **/etc/postfix/main.cf** and add the following lines to the end of the file.
``` sh
myhostname = hostname.example.com

relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
``` 
The **myhostname** parameter is optional. If the hostname is not specified, Postfix will use the fully-qualified domain name of the Linux server.

Save the **main.cf** file and close the editor.

## Configure Postfix SASL Credentials
The Gmail credentials must now be added for authentication. Create a **/etc/postfix/sasl_passwd** file and add following line:
``` sh
 [smtp.gmail.com]:587 username:password
 ```
The username and password values must be replaced with valid Gmail credentials. The **sasl_passwd** file can now be saved and closed.

A Postfix lookup table must now be generated from the **sasl_passwd** text file by running the following command.
``` sh
postmap /etc/postfix/sasl_passwd
```
Access to the **sasl_passwd** files should be restricted.
``` sh
chown root:postfix /etc/postfix/sasl_passwd*
chmod 640 /etc/postfix/sasl_passwd*
```
Lastly, reload the Postfix configuration.
``` sh
systemctl reload postfix
```
## Test the Relay
Use the mail command to test the relay.
``` sh
echo "This is a test." | mail -s "test message" user@example.net
```
The destination address should receive the test message.

## Troubleshoot Delivery Issues
The maillog can be reviewed if the test message is not successfully delivered. Open another shell and run tail while performing another test.
``` sh
tail -f /var/log/maillog
```
If there are not enough details in the **maillog** to determine the problem, then the debug level can be increased by adding the following lines to the **/etc/postfix/main.cf**.
``` sh
debug_peer_list=smtp.gmail.com
debug_peer_level=3
```
The Postfix configuration must be reloaded after updating the **main.cf** file.
``` sh
systemctl reload postfix
```
Remember to remove the debug settings when testing is complete. The verbose logs can have a negative impact on server performance.
