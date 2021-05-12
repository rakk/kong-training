# How to setup proxy for minikube

## Install proxy

```bash
brew install tinyproxy
```

## Configure proxy

In file `/usr/local/etc/tinyproxy/tinyproxy.conf` 
or (on MacOS) `/usr/local/etc/tinyproxy/tinyproxy.conf`
edit:
```
Listen 192.168.0.1
?
```
to
```
Listen 0.0.0.0
```
and edit:
```
Allow 127.0.0.1
```
to
```
# Allow 127.0.0.1
```

## Running proxy

Run proxy:
```
tinyproxy
```

## Very if proxy works

```bash
curl --proxy http://192.168.64.1:8888 https://example.com
```
expected output:
```bash
<!doctype html>
<html>
<head>
    <title>Example Domain</title>
(...)
```

## Prepare minikube

```
export HTTPS_PROXY=https://192.168.64.1:8888
export HTTP_PROXY=http://192.168.64.1:8888
export NO_PROXY=localhost,127.0.0.1,192.168.64.0/24,192.168.99.0/24
```

Now you can run: `minikube start`
