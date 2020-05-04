# Install Kong Ingress Controller

Source: https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/deployment/eks.md

!!! Note this does not install Kong Enterprise !!!
Add this point you don't need Kong Enterprise.
//TODO: add list of plugin that require enterprise

Kong Enterprise Deployments:
https://github.com/Kong/kubernetes-ingress-controller/tree/master/docs/deployment

## Install Ingress Controller:
```bash
kubectl create -f https://bit.ly/k4k8s
```

Verify the contents of the namespace:
```bash
kubectl get all -n kong
``` 

## Verify the install

Grab the Proxy Hostname:
```bash
kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" service -n kong kong-proxy
```
!!! Note that this might take some time !!!

Verify the Ingress Controller is working:
```bash
$ curl -i $(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" service -n kong kong-proxy)
HTTP/1.1 404 Not Found
Date: Fri, 21 Jun 2019 17:01:07 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
Server: kong/1.1.2

{"message":"no Route matched with those values"}
```

[Move on to Kong Ingress SSL Termination](../cert-manager/README.md)