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

[Move on to installing the EKS cluster](eks-cluster/README.md)