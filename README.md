# Setup a EKS Cluster with Kong Ingress Controller
Learnings from setting up an AWS EKS Cluster with Kong Ingress Controller

## Prerequisites

### create IAM role for EKS cluster management
* open ```https://console.aws.amazon.com/iam/``` and choose _Roles_ => _create role_  
* choose _EKS_ service followed by _Allows Amazon EKS to manage your clusters on your behalf_  
* choose _Next: Permissions_
* click _Next: Review_
* enter a *unique* Role name, _eks-tutorial-role_ and click *_Create Role_*

### create keypair for ssh access to EC2 instance

* open EC2 dashboard ```https://console.aws.amazon.com/ec2```
* click _KeyPairs_ in left navigation bar under section "Network&Security"
* click _Create Key Pair_
* provide name for keypair, _eks-tutorial-keypair_ and click *_Create_*
* !! the keypair will be downloaded immediately => file *eks-tutorial-keypair.pem* !!

### create API Access key/-secret
* create key+secret via AWS console
  AWS-console => IAM => Users => <your user> => tab *Security credentials* => button *Create access key*


### Install AWS CLI

Follow install instructions: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html

Verify if the install works:
```bash
aws --version
```

  
### Install eksctl

Follow install instructions: https://github.com/weaveworks/eksctl

Verify if the install works:
```bash
eksctl version
```

### Install kubectl

Follow install instructions: https://kubernetes.io/docs/tasks/tools/install-kubectl/
Verify if the install works:
```bash
eksctl version
```

[Move on to installing the EKS cluster](eks-cluster/README.md)