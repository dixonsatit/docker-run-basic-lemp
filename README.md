# Docker run basic (LEMP)

Prepare project:
```
git clone  https://github.com/dixonsatit/docker-run-basic-lemp.git
```
cd docker-run-basic-lemp

### Create local doman (dev)

change the hosts file to point the domain to your server
- Windows: `c:\Windows\System32\Drivers\etc\hosts`
- Linux: `/etc/hosts`

Add the following lines:
```
127.0.0.1   app-frontend.dev
```

### Create Network

```
docker network create web
```

### Run mysql
```
docker run -d --name mysql -p 3306:3306 --network web --network-alias db -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=web -e MYSQL_USER=web -e MYSQL_PASSWORD=1234 -v $(PWD)/data:/var/lib/mysql mysql:8.0.1
```

### Run php-fpm
```
docker run -d --name php-fpm -p 9000:9000 --network web --network-alias php -v $(PWD)/app:/var/www/html php:7-fpm-alpine
```

### Run nginx
```
docker run -d --name web -p 80:80 -v $(PWD)/app:/var/www/html -v $(PWD)/conf.d/default.conf:/etc/nginx/conf.d/default.conf --network web nginx:1.13-alpine
```

### nginx config

default.conf
``` 
server {
   charset utf-8;
   client_max_body_size 128M;
   sendfile off;

   listen 80; ## listen for ipv4
   #listen [::]:80 default_server ipv6only=on; ## listen for ipv6

   server_name app-frontend.dev;
   root        /var/www/html/;
   index       index.php;

   location / {
       # Redirect everything that isn't a real file to index.php
       try_files $uri $uri/ /index.php$is_args$args;
   }

   # uncomment to avoid processing of calls to non-existing static files by Yii
   #location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
   #    try_files $uri =404;
   #}
   #error_page 404 /404.html;

   location ~ \.php$ {
       include fastcgi_params;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       fastcgi_pass   php:9000;
       try_files $uri =404;
   }

   location ~ /\.(ht|svn|git) {
       deny all;
   }
}

```

