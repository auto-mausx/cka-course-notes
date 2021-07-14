# K8s Services Overview

> In this section, we will examine Kubernetes Services. This lesson provides a brief overview of Kubernetes Services and gives you an idea of what to expect as you proceed through this section.

## Relevant Docs

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

---

## What Is a Service?

> Kubernetes **Services** provide a way to expose an application running as a set of Pods

> They provide an abstract was for clients to be aware of the application's Pods.

## Service Routing

> Clients make requests to a Service, which **routes** traffic to its Pods in a load-balanced fashion.

*Note: It essentually acts as a load-balancer for an application and distributes between pods.*

## Endpoints

> **Endpoints** are the backend entities to which Services route traffic. For a Service that routes traffic to multiple Pods, each Pod will have an endpoint associated with the Service.

*Note: One way to determine which Pod(s) a Service is routing traffic to is to look at that service's Endpoints.*
