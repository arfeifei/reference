DOCKER – HOW TO CREATE A CUSTOM DATABASE FROM DOCKER MYSQL IMAGE

###Problem:

I use the official mysql base image from Docker Hub to create mysql database. But by default it only creates one database. I want to create another database.

###Solution:

mysql image actually provides a way to do this. Reference the official doc: https://github.com/docker-library/docs/tree/master/mysql  Under “Initializing a fresh instance” section.

So we just need to create a .sql file with create database command, and mount this file as volume to docker-entrypoint-initdb.d folder inside mysql container.

This is my docker-compose.yml file:
```
version: ‘2′

services:
  db:
    image: mysql   # Pull mysql image from Docker Hub
    environment:   # Set up mysql database name and password
      MYSQL_DATABASE: development
      MYSQL_ROOT_PASSWORD: password
    ports:    # Set up ports exposed for other containers to connect to
      - "3306"
    volumes:  # Mount relative path source folder on host to absolute path destination folder on docker container
      - ./docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d
  web:
    build: .  # Look in the current directory for a Dockerfile and build images using it
    depends_on:    # Link web container to db container
      - db
    environment:  # Set up environment variables
      MYSQL_DATABASE: development
      MYSQL_ROOT_PASSWORD: password
      RAILS_ENV: docker
    ports:
      - "3000:3000"  # Forwards the exposed port 3000 on the container to port 3000 on the host machine
```

We create init.sql file inside docker-entrypoint-initdb.d folder, and use “volumes” to mount this folder to docker-entrypoint-initdb.d inside mysql container, so that mysql service will have access to all the files in docker-entrypoint-initdb.d folder on host. The content of init.sql file:
```
CREATE DATABASE test;
```
After we run
```
$docker-compose up
```
It will boot up db and web service. The db service will be a mysql container, which have two databases: develop and test.

[Reference](https://littletechblogger.wordpress.com/2016/03/02/docker-how-to-create-a-custom-database-from-docker-mysql-image/)