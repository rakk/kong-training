# kong-training
Documentation shows how to setup Kong on Minikube locally and understand the basic setup

## Preparation

Start minikube

```bash
minikube start
```

Create namespace:
```bash
kubectl create ns super-kong
```

## Installation ECHO

Install example app:
```
kubectl -n super-kong apply -f https://bit.ly/echo-service
kubectl -n super-kong get pod
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
kubectl -n super-kong get pod
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

apiVersion: apps/v1
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
        backend:
          serviceName: echo
          servicePort: 80
" | kubectl -n super-kong apply -f -
```

Let's make a curl call:
```bash
curl -k $KONG_PROXY/foo/bar
```

(expected) response:
```bash
TODO: add reponse here!
```

Let's make a POST curl call:
```bash
curl -k -X POST $KONG_PROXY/foo/bar
```

(expected) response:
```bash
TODO: add reponse here!
```
