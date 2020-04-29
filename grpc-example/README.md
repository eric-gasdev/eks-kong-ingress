# Test gRPC 

source: https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/guides/using-ingress-with-grpc.md

## Deploy test gRPC Service:
```bash
kubectl apply -f https://bit.ly/grpcbin-service
```

## Create Ingress Rule:
```bash
echo "apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: grpcbin
          servicePort: 9001" | kubectl apply -f -
```

//TODO: capture scripts

## Update the Ingress rule to specify gRPC as the protocol

!!! Default Kong only supports http & https !!!

Add annotation to ingress rule:
```bash
kubectl patch ingress demo -p '{"metadata":{"annotations":{"konghq.com/protocols":"grpc,grpcs"}}}'
```

//TODO: check if this can be done as a one shot with creating rule

## Update the upstream protocol to be grpcs
!!! Default Kong only supports http & https !!!

Add annotation to service:
```bash
kubectl patch svc grpcbin -p '{"metadata":{"annotations":{"konghq.com/protocol":"grpcs"}}}'
```

//TODO: check if this can be done as a one shot with creating service

## Test with grpcurl

Run the docker container with interactive shell:
```bash
docker run -it networld/grpcurl sh
```

From inside the container:
```bash
./grpcurl -v -d '{"greeting": "Kong Hello world!"}' -insecure <Your proxy hostname>:443 hello.HelloService.SayHello
```

## Clean up

Delete Ingress Rule, Service, Deployment
```bash
kubectl delete ingress demo
kubectl delete svc grpcbin
kubectl delete deployment.apps/grpcbin
```

Verify that that no pods are running:
```bash
kubectl get po
```