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

Check the Kubernetes version on the AWS EKS Console: e.g. Kubernetes 1.15

Get the autoscaler image version:  
* open https://github.com/kubernetes/autoscaler/releases and get the latest release version matching your Kubernetes version
* E.g. 1.15.6 -> eu.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.15.6

edit deployment and set your EKS cluster name:

```bash
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```
Update the config using vi commands:
* ```eu.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.15.6```  
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
kubectl get po -o wide
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

!!! Note this takes some time, so you will see pods in Pending and other states !!!
Check pods:
```bash
$ kubectl get po -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP               NODE                                           NOMINATED NODE   READINESS GATES
test-autoscaler-797bb4dc87-4m5wx   1/1     Running   0          3m21s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-4tmrf   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-7mrb6   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-7v2lc   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-879dl   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-9g75s   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-9wf5p   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
test-autoscaler-797bb4dc87-9x4b5   1/1     Running   0          2m16s   xxx.xxx.xx.xx    ip-xxx-xxx-x-xx.eu-west-1.compute.internal     <none>           <none>
...
```
We see we have 30 pod running as expected. 

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
kubectl get po -o wide
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
//TODO: verify correct grep statement: changed since 1.15

### Clean up the Deployment
First list all resources:
```bash
kubectl get all
```

There you see the deployment name and we can delete it:
```bash
kubectl delete -f test-autoscaler-deployment.yaml
```

We can verify that the deployment and all his pods are deleted:
```bash
kubectl get all
```

[Move on to installing Kong Ingress Controller](../kong-ingress/README.md)