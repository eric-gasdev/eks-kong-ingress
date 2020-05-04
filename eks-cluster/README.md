# Create EKS cluster

Learnings setting up an EKS cluster that runs application workloads on Fargate with Kong Ingress Controller.

* Going full Fargate not possible
    * Error during Kong Ingress installation: LoadBalancer: Error registering targets for load balancer
    * See: https://discuss.konghq.com/t/aws-eks-fargate-with-kong-ingress-controllar-external-ip/5863
* kong namespace needs to be run on a nodegroup for NLB support
* kube-system namespace on a nodegroup for the autoscaler 

# Deleting EKS Cluster
!!! Running an EKS cluster is quite costly, so take down the stack as soon as possible !!!

```bash
eksctl delete cluster -f eks-cluster.yaml
```


## Create the EKS cluster
```bash
eksctl create cluster -f eks-cluster.yaml
```
!!!This takes aprox. 15 mins to setup and will configure kubectl!!!

Sets up an EKS cluster:
- name: spot-autoscale
- region: eu-west-1
- k8s version: 1.15
- 1 nodegroup:
    - name: ng-spot-autoscale
    - spot instances
    - autoscale profile
    - cert manager profile
    - 3 AZ: eu-west-1a, eu-west-1b, eu-west-1c
        
!!! Uses spot instance for cost saving, this needs to change in PRD !!!

## Verify the cluster is running

```bash
eksctl get cluster
```

## Verify kubectl is working

```bash
kubectl get nodes
```
We are now ready to interact with our EKS Cluster


[Move on to auto scaling the cluster](../cluster-autoscaler/README.md)
