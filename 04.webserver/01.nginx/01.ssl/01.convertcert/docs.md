---
title: Convert pfx to crt/key
taxonomy:
    category: docs
---
For the certificates (including CA and root certificates), just run the following command:
```
openssl pkcs12 -in domain.pfx -clcerts -cacerts -nokeys -out domain.crt
```
For the key:
```
openssl pkcs12 -in domain.pfx -nocerts -nodes -out domain.key
```