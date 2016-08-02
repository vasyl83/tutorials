---
title: HTPC reverse proxy
taxonomy:
    category: docs
---

Most of the following can be used as a subdomain or subfolder. To create a subdomain you must create a new server block, for the subfolder simply paste the location block into your existing server block and adjust as needed:

For syncthing:
```
location /syncthing/ {
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_pass              http://192.168.0.5:8384/;
        }
```
For Guacalome:
```
location /guac/ {
        proxy_pass http://192.168.0.5:9001/guacamole/;
        access_log off;
        proxy_set_header Accept-Encoding "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_cookie_path /guacamole/ /guac/;
        }
```

The `proxy_cookie_path /guacamole/ /guac/;` is only needed if your subfolder is different from the default one, if you omit it, it will still work but may take longer to load the page.