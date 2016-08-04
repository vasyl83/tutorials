---
title: SSL
taxonomy:
    category: docs
---
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) has a very thourough guide on how to setup [Let's Encrypt](https://letsencrypt.org/) certificate to secure your site. 

Also it is worth mentionning that subdomains can be added to a let's encrypt certificate and used with a separate server block. just make sure that root is pointed to the same folder for all subdomains (and the main domain) and add the following location block to make sure that auto renew of certificates doesn't fail:
```
location ~ /.well-known {
                allow all;
        }
```