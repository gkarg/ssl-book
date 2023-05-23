# What is SSL and how do I turn it off?

Definitive guide to SSL.

Contributions welcome, under CC-BY-SA license.

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

## AWS

`aws --no-verify-ssl s3 cp filename s3://bucketname/`

## Python

### In your own code

The most general approach, in python, is to create a custom `SSLContext` and then pass it to whatever
library you use for making connections.

```python
import ssl
import urllib.request

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

with urllib.request.urlopen("https://example.org", context=ctx) as u:
    data = u.read()
```

This also works with asyncio `create_connection`, via the `ssl` argument:

```python
import asyncio
import ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

connection = asyncio.get_running_loop().create_connection(..., ssl=ctx)
```

Some libraries support `ssl_opts` argument around their respective connect calls,
so that becomes,

```python
import ssl

import somelibrary

connection = somelibrary.connect(
   "https://example.org", 
   ssl_opts={"check_hostname": False, "verify_mode": ssl.CERT_NONE}
)
```

Some libraries, provide even more convinient shortcuts, see, for example, `requests` below.

#### requests

In requests, `verify=True` can be passed to `get`/`post`/`request`/etc methods, as well
as to the `Session` object, which can later be used to issue all your requests.

```python
import requests
from urllib3.exceptions import InsecureRequestWarning

# Optionally, suppress urllib3 warning
requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)

requests.get("https://example.org", verify=False)
```

See [here](https://stackoverflow.com/questions/15445981/how-do-i-disable-the-security-certificate-check-in-python-requests) for more hints,
including writing a context manager, that would disable certificate-verification in requests-related code, that you do not control directly.
