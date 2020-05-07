# SSL Termination on Ingress with cert-manager 

The k8s cert-manager will allow for SSL termination on the Ingress level.

See: https://cert-manager.io/docs/

We will implement with Let's Encrypt in combination with Route53.

Make sure that Route53 is setup with the Hosted Zone for your domain.

Sources:
* https://cert-manager.io/docs/installation/kubernetes/
* https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/guides/cert-manager.md
* https://aws.amazon.com/blogs/containers/securing-eks-ingress-contour-lets-encrypt-gitops/

## Install cert-manager

Kubernetes 1.15+ use:
```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.3/cert-manager.yaml
```

For lower version of k8s see: https://cert-manager.io/docs/installation/kubernetes/

Verify the install:
```bash
$ kubectl get all -n cert-manager

NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-86478c5ff-mkhb9               1/1     Running   0          23m
pod/cert-manager-cainjector-65dbccb8b6-6dnjl   1/1     Running   0          23m
pod/cert-manager-webhook-78f9d55fdf-5wcnp      1/1     Running   0          23m

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/cert-manager-webhook   ClusterIP   10.63.240.251   <none>        443/TCP   23m

NAME                                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1         1         1            1           23m
deployment.apps/cert-manager-cainjector   1         1         1            1           23m
deployment.apps/cert-manager-webhook      1         1         1            1           23m

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-86478c5ff               1         1         1       23m
replicaset.apps/cert-manager-cainjector-65dbccb8b6   1         1         1       23m
replicaset.apps/cert-manager-webhook-78f9d55fdf      1         1         1       23m
```

## Setup Cluster Issuer with Let's Encrypt en Route53

!!! For this your EKS nodegroup needs to include the iam addon profile certManager !!!

Open the cluster-issuer.yaml and add your email address

Install:
```bash
kubectl apply cluster-issuer.yaml
```

## Setup DNS Records

Using Route53 create:
* A Record
    * proxy.domain.com
    * Pointing to the AWS ELB (Kong Ingress NLB)
* C Record
    * demo.domain.com
    * Pointing to proxy.domain.com
    
## Deploy a sample application to verify SSL Termination

Open verify-ssl-termination-deployment.yaml:
* Replace:
    * domain.com by your own domain name: e.g. mydomain.co.uk
    * demo-domain-com by your own domain info: e.g. demo-mydomain-co-uk
    
Deploy the service:
```bash
kubectl apply -f verify-ssl-termination-deployment.yaml
```

Verify:
```bash
$ curl https://demo.domain.com
Hostname: echoserver-747b5cfd7d-xn89x

Pod Information:
	node name:	ip-xxx-xxx-xx-xxx.eu-west-1.compute.internal
	pod name:	echoserver-747b5cfd7d-xn89x
	pod namespace:	default
	pod IP:	xxx.xxx.xx.xxx

Server values:
	server_version=nginx: 1.12.2 - lua: 10010

Request Information:
	client_address=xxx.xxx.xx.xx
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://demo.domain.com:8080/

Request Headers:
	accept=*/*  
	connection=keep-alive  
	host=demo.domain.com  
	user-agent=curl/7.64.1  
	x-forwarded-for=xxx.xxx.xx.xxx  
	x-forwarded-host=demo.upgrade-it.be  
	x-forwarded-port=8443  
	x-forwarded-proto=https  
	x-real-ip=xxx.xxx.xx.xxx  

Request Body:
	-no body in request-
```

## Clean up
```bash
kubectl delete -f verify-ssl-termination-deployment.yaml
```


[Move on to gRPC](../grpc-example/README.md)




