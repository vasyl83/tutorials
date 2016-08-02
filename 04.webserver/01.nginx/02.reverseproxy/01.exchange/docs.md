---
title: MS Exchange 2016
taxonomy:
    category: docs
---

To proxy Exchange with all it functions nginx needs `ngx_headers_more` module, Debian has that module in nginx-full or nginx-extra packages.

The setup is pretty straight forward, in this example `https://postmaster.ad.gontar.net` should point to the internal URL of the Exchange service you are trying to make available from the outside. `more_set_headers -s 401 ‘WWW-Authenticate: Basic realm=”gontar.net”‘` should point to either you external or internal address to access Exchange. In my case postmaster.ad.gontar.net is my internal exchange address and gontar.net is external. Paste the following inside `server` block:

```
location /owa {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/owa;
}

location /Microsoft-Server-ActiveSync {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/Microsoft-Server-ActiveSync;
}

location /ecp {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/ecp;
}

location /rpc {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/rpc;
}

location /Autodiscover {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/Autodiscover;
}

location /OAB {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/OAB;
}

location /ews {
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_pass_header Date;
        proxy_pass_header Server;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        more_set_input_headers 'Authorization: $http_authorization';
        proxy_set_header Accept-Encoding "";
        more_set_headers -s 401 'WWW-Authenticate: Basic realm="gontar.net"';
        proxy_pass https://postmaster.ad.gontar.net/ews;
}
```
Autodiscover can use either autodiscover.gontar.net or gontar.net/autodiscover, both will work as any client trying to connect to your Exchange server will try one then the other. For completion sake, enable both by creating an additionnal `server` block for autodiscover subdomain (or if you are using a Let's Encrypt certificate, juste request autodiscover subdomain to be added to your certificate and add autodiscover to server name string):

```
server {
        listen 80;
        server_name autodiscover.gontar.net www.autodiscover.gontar.net;

        return 301 https://autodiscover.gontar.net$request_uri;
}

server {
        access_log /var/log/nginx/autodiscover.access.log main;
        error_log  /var/log/nginx/autodiscover.error.log info;
        index index.html index.php;
        limit_req zone=gulag burst=200 nodelay;

        listen 443 ssl;
        server_name autodiscover.gontar.net;

        # SSL certs
        ssl on;
        ssl_session_cache shared:SSL:1m;
        ssl_stapling on;
        ssl_stapling_verify on;
        ssl_certificate /etc/certificates/autodiscover.gontar.net.crt;
        ssl_certificate_key /etc/certificates/autodiscover.gontar.net.key;

        location / {
                proxy_http_version 1.1;
                proxy_pass_request_headers on;
                proxy_pass_header Date;
                proxy_pass_header Server;

                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                more_set_input_headers 'Authorization: $http_authorization';
                proxy_set_header Accept-Encoding "";
                more_set_headers -s 401 'WWW-Authenticate: Basic realm="auto$
                proxy_pass https://postmaster.ad.gontar.net;
        }
}
```