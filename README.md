# Ingress Canary Test

## Pre installation

[Installation ingress-nginx by Helm chart](https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx)

## Ingress canary test

NGINX Ingress Controller uses the following annotations to enable canary deployments:

- nginx.ingress.kubernetes.io/canary-by-header
- nginx.ingress.kubernetes.io/canary-by-header-value
- nginx.ingress.kubernetes.io/canary-by-header-pattern
- nginx.ingress.kubernetes.io/canary-by-cookie
- nginx.ingress.kubernetes.io/canary-weight

### step1

kubectl apply deployment and two services

```
kubectl apply -f step1_deploy/echo-v1.yml
kubectl apply -f step1_deploy/echo-v2.yml
```

### step2

create ingress for default service

```
kubectl apply -f step2_ingress/ingress.yml
```

Verify that traffic is successfully routed:

```
$ curl -H "Host: canary.test.com" http://canary.test.com/echo
"echo-v1"
```

### step3

test canary deployments by header

```
kubectl apply -f step3_ingress_by_header/ingress.yml
```

Verify that traffic is successfully routed:

```
$ curl -H "Host: canary.test.com" -H "x-region: us" http://canary.test.com/echo
"echo-v1"
$ curl -H "Host: canary.test.com" -H "x-region: kor" http://canary.test.com/echo
"echo-v2"
```

### step4

test canary deployments by cookie

```
kubectl apply -f step4_ingress_by_cookies/ingress.yml
```

Verify that traffic is successfully routed:

```
$ curl -H "Host: canary.test.com" --cookie "my_cookie=always" http://canary.test.com/echo
"echo-v1"
$ curl -H "Host: canary.test.com" --cookie "other_cookie=always" http://canary.test.com/echo
"echo-v2"
```

### step5

test canary deployments by weight

```
kubectl apply -f step3_ingress_by_weight/ingress.yml
```

Verify that traffic is successfully routed:

```
$ for i in {1..10}; do curl -H "Host: canary.test.com" http://canary.test.com/echo; done
"echo-v1"
"echo-v1"
"echo-v1"
"echo-v2"
"echo-v1"
"echo-v1"
"echo-v1"
"echo-v1"
"echo-v2"
"echo-v1"
```

## Reference

[Configure canary deployment my mirantis](https://docs.mirantis.com/mke/3.5/ops/deploy-apps-k8s/nginx-ingress/configure-canary-deployment.html)
