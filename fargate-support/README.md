# Enabling Fargate for Applications

We want to add Fargate support to our EKS Cluster.

Our ambition is to run our microservice workloads on Fargate. 

## Add Fargate support to the existing EKS cluster

```bash
eksctl create fargateprofile -f fargate-profile.yaml
```

- 1 fargate profile:
    - name: fargate-default
    - selector 
        - namespaces: default
        - label: compute-engine: fargate
        
!!! Note only deployment in the default namespace with the label: compute-engine:fargate will be deployed on Fargate !!!

## Deploy a sample application to Fargate

Deploy an echo server:
```bash
kubectl apply -f test-fargate-deployment.yaml
```

We can see 1 pod is deployed on Fargate:
```bash
kubectl get pods -o wide          
```

Call the echo server:
```bash
$curl -i $(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" service -n kong kong-proxy)/echo
 HTTP/1.1 200 OK
 Transfer-Encoding: chunked
 Connection: keep-alive
 Server: BaseHTTP/0.6 Python/3.5.0
 Date: Tue, 28 Apr 2020 21:40:47 GMT
 X-Kong-Upstream-Latency: 2
 X-Kong-Proxy-Latency: 1
 Via: kong/2.0.3
 ...
```

Clean up:
```bash
kubectl delete -f test-fargate-deployment.yaml
```