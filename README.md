# Docker-compose-with-wordpres-php-mysql-nginx
Docker-compose with wordpres-php-mysql-nginx

# Description

WordPress is a free and open-source Content Management System (CMS) built on a MySQL database with PHP processing. Thanks to its extensible plugin architecture and templating system, and the fact that most of its administration can be done through the web interface. 
Running WordPress typically involves installing a LAMP (Linux, Apache, MySQL, and PHP) or LEMP (Linux, Nginx, MySQL, and PHP) stack, which can be time-consuming. However, by using tools like Docker and Docker Compose, you can simplify the process of setting up your preferred stack and installing WordPress. Instead of installing individual components by hand, you can use images, which standardize things like libraries, configuration files, and environment variables, and run these images in containers, isolated processes that run on a shared operating system.

# Pre-Requests
A server instaled with docker and docker-compose

Iam using my EC2 instance for this project

Here we go! 

# Docker installation
~~~
sudo yum install docker -y
sudo systemctl restart docker.service
sudo systemctl enable docker.service
~~~

# docker-compose installation
~~~
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo usermod -a -G docker ec2-user
~~~

# Checking Docker-compose version

~~~
docker-compose version

docker-compose version 1.29.2, build 5becea4c
docker-py version: 5.0.0
CPython version: 3.7.10
~~~

# Building the code

Iam building the code inside a project directory called "My-project". 

Proceeding to create  a file  "docker-compose.yml" inside the project directory. in this file, we are bulding the codes to create containaers (for wordpress,mysql,nginx) and 2 volumes (for wordpress,mysql) then the network for enabling the connection between the containers.

~~~


version: '3'

services:

  database:

    image: mysql:5.7
    container_name: mysql
    restart: always
    networks:
      - wp-net
    volumes:
      - mysql-volume:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: mysqlroot123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:

    image: wordpress:php7.4-fpm-alpine
    container_name: wordpress
    restart: always
    networks:
      - wp-net
    volumes:
      - wp-volume:/var/www/html
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress


  nginx:

    image: nginx:alpine
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    networks:
      - wp-net
    volumes:
      - wp-volume:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./wordpress.jibin.online.crt:/var/ssl/wordpress.jibin.online.crt
      - ./wordpress.jibin.online.key:/var/ssl/wordpress.jibin.online.key

volumes:
  mysql-volume:
  wp-volume:

networks:
  wp-net:
~~~



# Now, proceeding to create file "nginx.conf" to do nginx reverse proxy.


~~~

server {
  listen 80;
    return 301 https://wordpress.jibin.online;

}
server {
 #  listen 80;
 listen 443 ssl;

  server_name wordpress.jibin.online;

   ssl_certificate /var/ssl/wordpress.jibin.online.crt;
  ssl_certificate_key  /var/ssl/wordpress.jibin.online.key;

  root /var/www/html;
  index index.php;

  if ($request_method = POST) {
      set $cache_uri 'null cache';
  }
  if ($query_string != "") {
      set $cache_uri 'null cache';
  }

 location / {
  try_files $uri $uri/ /index.php?$args;
}

#PHP scripts to FastCGI server listening on wordpress:9000
location ~ \.php$ {
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  fastcgi_pass wordpress:9000;
  include fastcgi_params;
  fastcgi_index index.php ;
  include fastcgi_params;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  fastcgi_param SCRIPT_NAME $fastcgi_script_name;
}
}

~~~

Now, nginx configuration also been created. 

Iam using the domain "wordpress.jibin.online" for this project. I have issued a free SSL from "zero SSL" for the domain.

~~~
https://www.sslforfree.com/
~~~

We will be able to issue free SSL for 90 days from here. I issued and did DNS validation through adding the provided CNAME record to route53 in which my domain's DNS is managed by. I downloaded the SSL cirtificate and kept inside my project directory.

~~~
 [ec2-user@ip-172-31-17-22 my-project]$ ll
total 20
-rw-rw-r-- 1 ec2-user ec2-user 1132 Apr  9 08:32 docker-compose.yml
-rw-rw-r-- 1 ec2-user ec2-user  867 Apr 10 18:46 nginx.conf
-rw-rw-r-- 1 ec2-user ec2-user 4740 Apr 10 18:45 wordpress.jibin.online.crt
-rw-rw-r-- 1 ec2-user ec2-user 1680 Apr 10 18:44 wordpress.jibin.online.key
~~~


# Running the compose now. (commands must be executed inside the project directory)

## To check the syntax 
~~~
docker-compose config
~~~

## To run the compose
~~~
docker-compose up -d
~~~

## To view the containers created
~~~
docker container ls
~~~

## To view the network created
~~~
docker network ls
~~~

## To view the volume created
~~~
docker volume ls
~~~

## To remove all created containers and resources (The volume will be needed to remove manually)
~~~
docker-compose down
~~~


# Checking the containers created

~~~
docker container ls
CONTAINER ID   IMAGE                         COMMAND                  CREATED        STATUS        PORTS                                                                      NAMES
9de092b0226f   mysql:5.7                     "docker-entrypoint.s…"   2 hours ago    Up 2 hours    3306/tcp, 33060/tcp                                                        mysql
634e376503d4   nginx:alpine                  "/docker-entrypoint.…"   2 hours ago    Up 2 hours    0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   nginx
9f8c07e61999   wordpress:php7.4-fpm-alpine   "docker-entrypoint.s…"   2 hours ago    Up 2 hours    9000/tcp                                                                   wordpress

~~~


Once the compose ran, can proceed to install and configure the wordpress by calling the domain name "wordpress.jibin.online" on browser

## Website

![image](https://user-images.githubusercontent.com/100774483/162638843-2d406d54-4208-48ea-b1b1-a7727dcdb1b8.png)


# Conclusion

In this tutorial we discussed how to create and run wordpress with nginx reverse proxy through docker compose. The goal is to get you started on using this methode As this is easy and secure.

