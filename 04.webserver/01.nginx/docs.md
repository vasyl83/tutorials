---
title: Nginx
taxonomy:
    category: docs
---

After having used Apache, lighttpd, IIS and nginx I always returned to nginx, it always felt easier and more intuitive for me. For that reason I decided to use it exclusively, whenever possible. 

As covered in [Docker chapter](../../fileserver/os/docker/), I use docker extensively. nginx is no exception. For security, isolation and backup reasons, it is always better to run every app and every separate website in a different container or vm. The problem arises when we want to make those apps and services available from the ouside or even on our own network. Since every container cannot be run on the same port and remembering on which port everything is running is hard, we will install nginx directly on the OS in order to use it to proxy everything to the containers. This way we will be only exposing ports 80 and 443 to the outside, and we will make all the services available either through a subdomain or a subfolder on a domain. So to access SickRage instead of typing http://debian:8081/home one can simply type http://debian/sickrage.

Additionnal advantage of doing it this way is the fact that we will only need to secure one instance of nginx, so only one place where to put our certificates, and reduced attack surface from the outside!

#####Installation
There are several different packages of Nginx for Debian: nginx, nginx-full and nginx-extras the differences are explained [here](https://wiki.debian.org/Nginx). If Microsoft Exchange is used for email and is not directly exposed to the outside, nginx [headers_more module](https://github.com/openresty/headers-more-nginx-module) is required to correctly proxy all of the Exchange functions. That limits the package selection to only nginx-extras. To install it simply run `apt-get install nginx-extras`. Once installed, simply point your browser to the ip of the box where nginx was installed. If you see the default page, you are done. If nothing shows, check `/var/log/nginx/` to see if there is a problem.

#####Configuration
Instead of going line by line, here is the final nginx.conf for my server. Nginx site has a very good [wiki](https://www.nginx.com/resources/wiki/).
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
worker_rlimit_nofile 40000;

events {
        worker_connections  8096;
        multi_accept        on;
        use                 epoll;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        client_max_body_size 0;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;
		
        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites/*;
}
```
The is no SSL setting because I prefer to manage them inside `server` blocks. I also added `include /etc/nginx/sites/*;` this way I can have per site config file. I also skipped site-available/site-enabled model because I found myself always forgetting to create symlinks to my newly created `server` blocks.