---
layout: post
title: Using Docker to generate SSL artifacts
# image: http://openssl.com/images/openssl-logo.png
tags: [openssl, docker]
published: false
---

[https://github.com/paulczar/omgwtfssl](https://github.com/paulczar/omgwtfssl)

* use environment vars to override things
* SSL_DNS for SAN (Subject Alternative Names) certs
* use a volume to capture the SSL artifacts to a local directory (and push to source control)

```
docker run -e SSL_SUBJECT="example.com" -e SSL_DNS="*.example.com,*.something.com" -v /Users/pdaugavietis/:/certs  paulczar/omgwtfssl
```

```
openssl x509 -text -noout -in cert.pem
openssl x509 -text -noout -in ca.pem
openssl req -text -noout -in key.csr
```

[https://stackoverflow.com/a/14434262](https://stackoverflow.com/a/14434262)
