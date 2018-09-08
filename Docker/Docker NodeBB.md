Use docker-compose create website

It's being a week since I use docker to create my first peronal website.

I have to say **docker is awesome**!

Here is the complete source code about how to setup and use it in my github repostory which you can checkout from here:
>[https://github.com/arfeifei/www](https://github.com/arfeifei/www)

Enjoy!

##Some hiccups when bumped:
>
####1.docker-compose.yml volumes
Need set volumes to local directory otherwise the user data will be gone when execute ***docker-compose rm -f*** remove old container 

####2.NodeBB config.json
>1.url need match the site name
2.socket.io need allow all the origins
```
{
    "url": "http://arfeifei.no-ip.biz/forum",
    ...
    "socket.io": {
        "origins": "*:*"
    }
} 
        
```
####3.wordpress url must match domain path
Nginx.conf proxy_pass http://arfeifei.no-ip.biz to http://wordpress-app:8000.
login to wordpress by http://arfeifei.no-ip.biz/wp-login.php 
>Change wordpress (URL) & site (URL) to http://arfeifei.no-ip.biz

