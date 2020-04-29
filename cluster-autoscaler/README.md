# Setup Cluster Autoscaling

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

## Update autoscaler to match cluster and version 

Check the Kubernetes version on the AWS EKS Console: Kubernetes 1.14
Get the autoscaler image version:  
    - open https://github.com/kubernetes/autoscaler/releases and get the latest release version matching your Kubernetes version: 1.14.8

edit deployment and set your EKS cluster name:

```bash
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```
Update the config using vi commands:
* ```image=k8s.gcr.io/cluster-autoscaler:1.14.8```  
* ```- --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/spot-autoscale```

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

[Move on to installing Kong Ingress Controller](../kong-ingress/README.md)