# Microservices with consul - Kubernetes Example

This repository contains a simple example of how to use consul with kubernetes to create a service discovery system and
service mesh using consul connect. The Microservices are a nice and small example of services that can easily be run on 
kubernetes. The credit for these belongs to [Eberhard Wolff](https://github.com/ewolff) and his 
[microservices-kubernetes repo](https://github.com/ewolff/microservice-kubernetes)

This repository is a fork of the original repository with the addition of consul and consul connect to the microservices.
I have replaced the k8 manifest files with custom ones that include consul and consul connect. I have also added a
consul setup guide to help you get started with consul on kubernetes.

## What is consul
HashiCorp Consul is a service networking solution that enables teams to manage secure network connectivity between 
services and across on-prem and multi-cloud environments and runtimes. Consul offers service discovery, service mesh, 
traffic management, and automated updates to network infrastructure devices. You can use these features individually or 
together in a single Consul deployment.


## How it works
Consul provides a control plane that enables you to register, query, and secure services deployed across your network. 
The control plane is the part of the network infrastructure that maintains a central registry to track services and 
their respective IP addresses. It is a distributed system that runs on clusters of nodes, such as physical servers, 
cloud instances, virtual machines, or containers.

Consul interacts with the data plane through proxies. The data plane is the part of the network infrastructure that 
processes data requests.


## Deployment Guides
I have broken down the deployment into multiple guides to make it easier to follow. The guides will be used to build 
various layers of knowledge to help understand how we can take microservices that are unaware of consul and still 
protect and manage them through the service mesh.

We will first install consul and then deploy the microservices outside of consul. We will go on to redeploy them behind
service mesh proxies without making any changes to the microservices themselves. We will then look at how we can manage
traffic between the microservices and finally how we can observe the microservices.

* [Consul](consul-setup/README.md) - How to install consul on kubernetes
* [vanilla microservices](microservices-vanilla-k8/README.md) - How to deploy the microservices on kubernetes. These are the 
  microservices without consul.
* [service mesh microservices](microservices-consul-k8/README.md) - How to deploy the microservices on kubernetes with consul 
  connect. These are the microservices utilising the service mesh.
* [Traffic Management](traffic-management/README.md) - How to shape traffic between the microservices using consul connect.
* [Observability](observability/README.md) - How to observe the microservices using consul connect.

## Transparent Proxy
By default, the Consul service mesh on kubernetes runs in 
[transparent proxy mode](https://developer.hashicorp.com/consul/docs/k8s/connect/transparent-proxy). This mode forces 
inbound and outbound traffic through the sidecar proxy even though the service binds to all interfaces. Transparent 
proxy infers the location of upstream services using Consul service intentions, and also allows you to use Kubernetes 
DNS as you normally would for your workloads.

The benefit of transparent proxy mode is that it requires no changes to your application code or configuration. It also 
ensures that regardless of the interface the traffic arrives on it will go through the sidecar proxy. This allows it to 
always be encrypted and authenticated. When using the transparent proxy mode, consul will infer all the routes from 
service intentions. This means you don't need to explicitly configure upstreams.

When the transparent proxy is disabled you will need to configure the upstreams using pod annotations. You should also 
ensure that services are only listening on localhost to prevent direct access to the service. Also when services want
to talk to other services they will need to use the localhost of the proxy port to ensure it goes through the proxy and
is encrypted and authenticated.

When the transparent proxy is disabled routes are inferred from the upstreams defined in the pod annotations. This
means that you need to define the upstreams for the service in the pod annotations. For example, if you have an nginx
service that needs to talk to a frontend and public-api service you would define the upstreams like so:

```yaml
      annotations:
        consul.hashicorp.com/connect-inject: "true"
        consul.hashicorp.com/connect-service-upstreams: "frontend:3000, public-api:8080"
```
