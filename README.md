# :hammer:  API Gateway

## Goals/Problem statement

With the current trend of business needs, Data has to be obtained from one or more services to solve a 'Single Problem' or to get a 'Single Logical Response'. And hence occurs the need to have a Secure platform where these services are managed and deployed on-demand basis with a Secure exchange of Data.

ATOM API Gateway is a single point of contact through which clients/end-users interact with a Service (implemented with a bunch of Microservices) via Secure API. In other words, Clients interact with ATOM via Secure API. It hides the on-going changes happen within the Microservices and exposes a concrete API interface to the external world.

API gateway acts as an interface between these functional blocks:

* External facing API
* Microservices that provide the implementation for those APIs. These can be Containers, Servers listening on some ports.

## API Gateway Architecture

<p align="center">
  <img src="apigw-arch.png" />
</p>

We can go over major usecases of API Gateway in the following sections.

## API Gateway Usecases

### Security (Reverse Proxy)

API Gateway acts as a Proxy by providing safety net to the backend applications by abstracting the internal network.

* SSL Termination.
* Clients are independent of service lifecycle. Any config changes happen within the micro-services are not exposed to the end-clients. Any service going up and down is abstraced to the end-clients.
* It can reject HTTP Requests from a specific IP or restrict a number of open connections for a set of IPs.
* It can provide DDOS(Distributed Denial of Service). For ex., based on the health of each backend service, Requests can be denied or throttled.
* API Gateway can also compress the outgoing responses to increase the response-latency, thus would increase the scale.

### Manage Route Table

One of the functionalities of API Gateway is to route all the incoming Requests to its registered corresponding Service. It does this via 'Specific URL Match'.

For ex., Say if configured URLs

* /dashboard/*           - Handler1
* /dashboard/api/*       - Handler2
* /dashboard/api/things  - Handler3

Say, the incoming request is '/dashboard/api/things', it has to be matched first, hence Handler3 gets invoked. Say if that is not present, the next match would be '/dashboard/api/*', hence Handler2 will get invoked, and if that is also not present, then Handler1 gets invoked.

### Service Discovery

Each Service as and when it comes up, it can store its 'Set of URLs plus Config' in a distributed database say consul. API Gateway will monitor this database, any change to it will trigger service bringup/shutdown or dynamically push the config on a running Service. The Config itself can be in the form of a File that has etcd config, in the (key, value) form. 

API Gateway will populate its Route table with the URLs provided by these newly discovered Sevices and can choose to its Health and Metrics. This is again based on Applications config, if it needs to be Watched/Monitored, Metrics.

### Load Balancer

Many Servers of the same kind may be running at the backend to address the volume of incoming Requests. In that case, based on the config of Resource availability, say for ex., if there are multiple instances of the same service is running, one which has higher resource availability should be utilized more compared to the one that is comparitively less. if all the instances have the same resources allocated, Requests can be sent on round-robin. Each Service can have its own config for Load Balancer, for ex. one with Weighted Round Robin 

### Map Reduce

Say if Request needs response from one or more Services, the request has to be de-muxed into those Services, gather the corresponding responses, aggregate them and send a Single Response back. For ex.,

* REST API Request —>[API Gateway]—>[Compose/Consolidate Request/Requests to/From different Svcs, say Svc1, Svc2, ...SvcN]

### Schema Validation

Each Service can register its own Schema with API Gateway. No monolothic Schema, independent Schemas for each service. 

### Hot Reload

API Gateway should not get restarted if there is a change to the existing Config or New config is pushed. 

### Composition and Aggregation

* Req->APIGW[Validate Schema, say S]->appSvc[Expects different Request, say Req’ as Schema changed say S’]
* REST->APIGW[Validate Schema S + Change Incoming Request that matches S’ = Req']->appSvc[Receives Req’ as expected]

Are we going to support this in the first phase? Say for ex., Can we assume North Bound Schema would be same as South Bound or the Middleware can change the Incoming Request

### POST Data [CRUD]

API gateway should support the REST API - Create, Read, Update and Delete Operations

### Monitor Health and Metrics

API Gateway should monitor the Health of the Microservices. Each service can register itself with API Gateway get its health monitored. Say for ex., it can send Heart-beats on a regular intervals and if there is no response from a Service, it can restart that particular service for certain number of retries.

### Protocol Translation

API Gateway can accept REST and do REST via HTTP on either directions between the Clients and the Microservices or it can recieve HTTP request and in turn convert into other Protocols say for ex., GRPC based on the Schema defined by the individual Service. (again for ex.,). That way, we can support third party applications written. Some of the Protocols that we can support would be GRPC. 

To summarize:
API Gateway also can support the same protocols on either directions. Say for ex.,

* 'Recieve HTTP Requests' and 'Redirect/Send HTTP Requests' to the backends.
* 'Receive GRPC Requests' and 'Send GRPC' Requests' to the backends.
* 'Receive HTTP Requests' Convert into 'GRPC Requests' and send to the backends.

### Latency & Throughput 

Things that can be done to improve the Latency and throughput.

* SSL Termination at the API Gateway, so Requests are decrypted and Responses are encrypted at this layer, as encryption and decryption is very expensive.
* Caching the Responses before sending to clients, so that if the same Request is repeated multiple times, the responses can be retrieved from the Cache instead of handing over the Requests to the backend, - reducing the round-trip delay.
* It can also Compress the outgoing Responses by increasing the latency (ie reducing the round-trip delay). The goal is to achieve the Latency and Throughput close or near to Industry standard
