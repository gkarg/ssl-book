# What is SSL and how do I turn it off?

Definitive guide to SSL

## Introduction

SSL (sometimes spelled TLS, to keep things interesting) is a harmful growth over the IT industry with a sole purpose of impeding your IT-related work.

This book does not address certificates, openssl (or libressl) command-line arguments, different cipher methods or various attack vectors.

Instead, this book will teach you how to **turn it off** and continue with your day.

## cli tools

### curl

```
curl: (60) SSL certificate problem: Invalid certificate chain
More details here: https://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

This is very helpful, so the option you're looking for is `-k` or `--insecure`.

And viola `curl -k -O https://example.org/my_file.tar.gz` should do the trick!

### wget

The option you are looking for is `--no-check-certificate`, e.g.:

`wget --no-check-certificate https://example.org/my_file.tar.gz`

## Kubernetes

`kubectl --insecure-skip-tls-verify=true get pods`

## Python

### In your own code

#### requests

```python
import requests
from urllib3.exceptions import InsecureRequestWarning

# Suppress only the single warning from urllib3 needed.
requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)

# Set `verify=False` on `requests.post`.
requests.post(url='https://example.com', data={'bar':'baz'}, verify=False)
```

See [here](https://stackoverflow.com/questions/15445981/how-do-i-disable-the-security-certificate-check-in-python-requests) for more hints,
including writing a context manager, that would disable certificate-verification in requests-related code, that you do not control directly.
