# kong-training
Documentation shows how to setup Kong on Minikube locally and understand the basic setup

## Preparation

Start minikube

```bash
minikube start
```

If you see info that there is an issue with connection to gcr.io
check how to setup [proxy for minikube](proxy).

Create namespace:
```bash
kubectl create ns super-kong
```

## Installation ECHO

Install example app:
```
kubectl -n super-kong apply -f https://bit.ly/echo-service
kubectl -n super-kong get pod -w
```

## Installation KONG

Adding Kong helm chart:
```bash
helm repo add kong https://charts.konghq.com
helm repo update
```

Install kong:

```bash
helm install super kong/kong -f values/kong.values.yaml --namespace super-kong
kubectl -n super-kong get pod -w
```

Check out [values/kong.values.yaml](values/kong.values.yaml) for details.

```bash
export KONG_PROXY=$(minikube service -n super-kong super-kong-proxy --url | sed 's/http:/https:/')
echo "KONG_PROXY=${KONG_PROXY}"
```

## Basic Kong example

Creating `demo` Ingress:
```bash
echo "

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
  annotations:
    kubernetes.io/ingress.class: super-kong
spec:
  rules:
  - http:
      paths:
      - path: /foo
        pathType: Prefix
        backend:
          service:
            name: echo
            port:
              number: 80

" | kubectl -n super-kong apply -f -
```

Let's make a GET curl call:
```bash
curl -k $KONG_PROXY/foo/bar
```

(expected) response:
```bash
(...)
Request Information:
	client_address=172.17.0.2
	method=GET
	real path=/foo/bar
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.64.6:8080/foo/bar

Request Headers:
	accept=*/*  
	connection=keep-alive  
	host=192.168.64.6:32421  
	user-agent=curl/7.64.1  
	x-forwarded-for=172.17.0.1  
	x-forwarded-host=192.168.64.6  
	x-forwarded-path=/foo/bar  
	x-forwarded-port=443  
	x-forwarded-proto=https  
	x-real-ip=172.17.0.1  

Request Body:
	-no body in request-
```

Let's make a POST curl call:
```bash
curl -k -X POST $KONG_PROXY/foo/bar
```

(expected) response:
```bash
(...)
Request Information:
	client_address=172.17.0.2
	method=POST
	real path=/foo/bar
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.64.6:8080/foo/bar

Request Headers:
	accept=*/*  
	connection=keep-alive  
	host=192.168.64.6:32421  
	user-agent=curl/7.64.1  
	x-forwarded-for=172.17.0.1  
	x-forwarded-host=192.168.64.6  
	x-forwarded-path=/foo/bar  
	x-forwarded-port=443  
	x-forwarded-proto=https  
	x-real-ip=172.17.0.1  

Request Body:
	-no body in request-
```

Hurray! Our Kong is working! In next step we will extend our configuration.

## Let's block POST request by using KongIngress

Let's add KongIngress:
```bash
echo "

apiVersion: configuration.konghq.com/v1
kind: KongIngress
metadata:
  name: demo
  annotations:
    kubernetes.io/ingress.class: super-kong
route:
  methods:
  - GET
  strip_path: true

" | kubectl -n super-kong apply -f -
```

The GET request is still working fine:
```bash
curl -k $KONG_PROXY/foo/bar
```

(expected) response:
```bash
(...)
Request Information:
	client_address=172.17.0.2
	method=GET
	real path=/foo/bar
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.64.6:8080/foo/bar

Request Headers:
	accept=*/*  
	connection=keep-alive  
	host=192.168.64.6:32421  
	user-agent=curl/7.64.1  
	x-forwarded-for=172.17.0.1  
	x-forwarded-host=192.168.64.6  
	x-forwarded-path=/foo/bar  
	x-forwarded-port=443  
	x-forwarded-proto=https  
	x-real-ip=172.17.0.1  

Request Body:
	-no body in request-
```

But POST is not working anymore:

```bash
curl -k -X POST $KONG_PROXY/foo/bar
```

expected output:
```bash
{"message":"no Route matched with those values"}
```
