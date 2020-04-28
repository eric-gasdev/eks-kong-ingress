# Setup a EKS Cluster with Kong Ingress Controller
Learnings from setting up an AWS EKS Cluster with Kong Ingress Controller

## Prereq.

### create IAM role
* open ```https://console.aws.amazon.com/iam/``` and choose _Roles_ => _create role_  
* choose _EKS_ service followed by _Allows Amazon EKS to manage your clusters on your behalf_  
* choose _Next: Permissions_
* click _Next: Review_
* enter a *unique* Role name, _eks-tutorial-role_ and click *_Create Role_*

### create keypair

* open EC2 dashboard ```https://console.aws.amazon.com/ec2```
* click _KeyPairs_ in left navigation bar under section "Network&Security"
* click _Create Key Pair_
* provide name for keypair, _eks-tutorial-keypair_ and click *_Create_*
* !! the keypair will be downloaded immediately => file *eks-tutorial-keypair.pem* !!

### create API Access key/-secret
* create key+secret via AWS console
  AWS-console => IAM => Users => <your user> => tab *Security credentials* => button *Create access key*

# Deleting EKS Cluster
!!! Running an EKS cluster is quite costly, so take down the stack as soon as possible !!!

```bash
eksctl delete cluster -f eks-cluster.yaml
```

# Create EKS cluster with node groups

```bash
eksctl create cluster -f eks-cluster.yaml
```
!!!This takes aprox. 15 mins to setup and will configure kubectl!!!

## Sets up an EKS cluster:

- name: eks-test-cluster
- region: eu-west-1
    - 1 nodegroup:
        - name: scale-spot
        - spot instance
        - autoscale
        - 3 AZ: eu-west-1a, eu-west-1b, eu-west-1c

## Verify the cluster is running

```bash
eksctl get cluster
```

## Verify kubectl is working

```bash
kubectl get nodes
```
We are now ready to interact with our EKS Cluster

# Setup Autoscaling

## Deploy the autoscaler

Use the yaml file provided by Kubernetes itself:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

## Make sure that the autoscaler does not evict it's own pod

Add annotation to the deployment:
```bash
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```

## Fetch the latest autoscaler to match the running Kubernetes version (Optional)

Check the Kubernetes version on the AWS EKS Console: Kubernetes 1.14
Get the autoscaler image version:  
    - open https://github.com/kubernetes/autoscaler/releases and get the latest release version matching your Kubernetes version: 1.14.8

edit deployment and set your EKS cluster name:

```bash
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```
Update the config using vi commands:
* ```image=k8s.gcr.io/cluster-autoscaler:1.14.8```  
* ```- --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/eks-test-cluster```

## Verify autoscaler deployment

```bash
kubectl -n kube-system describe deployment.apps/cluster-autoscaler
```


## View cluster autoscaler logs

```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler
```

## Test the autoscaler

### Create a test deployment of nginx

```bash
kubectl apply -f test-autoscaler-deployment.yaml
```

Check pods:
```bash
kubectl get po
```

We see we have 1 pod running as expected

Check Spot Nodes:
```bash
kubectl get nodes -l instance-type=spot
```

We see that we have 1 node running as expected


### Scale the deployment out

Scale the number of replicas to 30:
```bash
kubectl scale --replicas=30 deployment/test-autoscaler
```

Check pods:
```bash
kubectl get po
```

We see we have 30 pod running as expected. 

!!! Note this takes some time, so you will see pods in Pending and other states !!!


Check Spot Nodes if all pod are running:
```bash
kubectl get nodes -l instance-type=spot
```

We see that we have multiple node running as expected

We can verify this in the logs:
```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "Expanding Node Group"
```

### Scale the deployment in

Scale the number of replicas to 1:
```bash
kubectl scale --replicas=1 deployment/test-autoscaler
```

Check pods:
```bash
kubectl get po
```

We see we have 1 pod running as expected. 

Check Spot Nodes if all pod are running:
```bash
kubectl get nodes -l instance-type=spot
```

We see that we have 1 node running as expected

!!! Note nodes will be scale down with a cooldown period. So this will take time!!!

We can verify this in the logs:
```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "removing node"
```

### Clean up the Deployment
First list all resources:
```bash
kubectl get all
```

There you see the deployment name and we can delete it:
```bash
kubectl delete deployment/test-autoscaler
```

We can verify that the deployment and all his pods are deleted:
```bash
kubectl get all
```


## Install Kong Ingress Controller

Source: https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/deployment/eks.md

!!! Note this does not install Kong Enterprise !!!
Add this point you don't need Kong Enterprise.
//TODO: add list of plugin that require enterprise

Kong Enterprise Deployments:
https://github.com/Kong/kubernetes-ingress-controller/tree/master/docs/deployment

Install Ingress Controller:
```bash
kubectl create -f https://bit.ly/k4k8s
```

Verify the contents of the namespace:
```bash
kubectl get all -n kong
```

Grab the Proxy Hostname:
```bash
kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" service -n kong kong-proxy
```
Verify the Ingress Controller is working:
```bash
$ curl -i <Your proxy hostname>
HTTP/1.1 404 Not Found
Date: Fri, 21 Jun 2019 17:01:07 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
Server: kong/1.1.2

{"message":"no Route matched with those values"}
```


## Test gRPC 

source: https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/guides/using-ingress-with-grpc.md

Deploy test gRPC Service:
```bash
kubectl apply -f https://bit.ly/grpcbin-service
```

Create Ingress Rule:
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

### Update the Ingress rule to specify gRPC as the protocol

!!! Default Kong only supports http & https !!!

Add annotation to ingress rule:
```bash
kubectl patch ingress demo -p '{"metadata":{"annotations":{"konghq.com/protocols":"grpc,grpcs"}}}'
```

//TODO: check if this can be done as a one shot with creating rule

### Update the upstream protocol to be grpcs
!!! Default Kong only supports http & https !!!

Add annotation to service:
```bash
kubectl patch svc grpcbin -p '{"metadata":{"annotations":{"konghq.com/protocol":"grpcs"}}}'
```

//TODO: check if this can be done as a one shot with creating service

### Test with grpcurl

Run the docker container with interactive shell:
```bash
docker run -it networld/grpcurl sh
```

From inside the container:
```bash
./grpcurl -v -d '{"greeting": "Kong Hello world!"}' -insecure <Your proxy hostname>:443 hello.HelloService.SayHello
```

### Clean up

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
