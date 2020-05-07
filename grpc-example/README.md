# Test gRPC 

## Deploy test gRPC Service:

Open grpc-example-deployment.yaml:
* Replace:
    * domain.com by your own domain name: e.g. mydomain.co.uk
    * demo-domain-com by your own domain info: e.g. demo-mydomain-co-uk

```bash
kubectl apply -f grpc-example-deployment.yaml
```
## Test with BloomRPC

* Import the proto from: https://raw.githubusercontent.com/marknazareno/grpc-spring-boot-demo/master/grpc-spring-boot-demo-proto/src/main/proto/helloworld.proto
* Select SayHello Method
* Fill in URL: demo.domain.com:443
* Click TLS: Select Server Certificate
* Fire request

## Test with grpcurl

Run the docker container with interactive shell:
```bash
docker run -it networld/grpcurl sh
```

From inside the container:
```bash
$ wget https://raw.githubusercontent.com/marknazareno/grpc-spring-boot-demo/master/grpc-spring-boot-demo-proto/src/main/proto/helloworld.proto
$ ./grpcurl -v -d '{"name": "from grpCurl"}' -proto helloworld.proto demo.domain.com:443 helloworld.Greeter.SayHello
Resolved method descriptor:
{
  "name": "SayHello",
  "inputType": ".helloworld.HelloRequest",
  "outputType": ".helloworld.HelloReply",
  "options": {
    
  }
}

Request metadata to send:
(empty)

Response headers received:
server: openresty
date: Mon, 04 May 2020 15:19:19 GMT
content-type: application/grpc
grpc-accept-encoding: gzip
x-kong-upstream-latency: 3
x-kong-proxy-latency: 0
via: kong/2.0.3


Response contents:
{
  "message": "Hello Kong"
}

Response trailers received:
(empty)
Sent 1 request and received 1 response
/ # ./grpcurl -v -d '{"name": "Kong"}' -proto helloworld.proto demo.domain.com:443 helloworld.Greeter.SayHello

Resolved method descriptor:
{
  "name": "SayHello",
  "inputType": ".helloworld.HelloRequest",
  "outputType": ".helloworld.HelloReply",
  "options": {
    
  }
}

Request metadata to send:
(empty)

Response headers received:
grpc-accept-encoding: gzip
x-kong-upstream-latency: 10
x-kong-proxy-latency: 1
via: kong/2.0.3
server: openresty
date: Mon, 04 May 2020 15:19:23 GMT
content-type: application/grpc


Response contents:
{
  "message": "Hello Kong"
}

Response trailers received:
(empty)
Sent 1 request and received 1 response

```

## Clean up
```bash
kubectl delete -f grpc-example-deployment.yaml
```

[Move on to Fargate Support](../fargate-support/README.md)