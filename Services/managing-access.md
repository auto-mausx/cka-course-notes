# Managing Access from Outside with K8s Ingress

> When running applications in Kubernetes, you will likely want outside entities to be able to interact with them. Kubernetes Ingress objects allow you to define how this access occurs. In this lesson, we will introduce and discuss Ingress, and we will create a basic Ingress for a Kubernetes Service.

## Relevant Docs

- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

## Lesson Reference

*Note: This lesson depends on objects that were created in the previous lesson. If you are following along, you will need to go through the previous lesson first.*

1. Create an ingress that maps to a Service:

```bash
vi my-ingress.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - path: /somepath
        pathType: Prefix
        backend:
          service:
            name: svc-clusterip
            port:
              number: 80
```

```bash
kubectl create -f my-ingress.yml
```

2. Check the status of the ingress:

```bash
kubectl describe ingress my-ingress
```

3. Update the Service to provide a name for the Service port:

```bash
vi svc-clusterip.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  type: ClusterIP
  selector:
    app: svc-example
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
```

```bash
kubectl apply -f svc-clusterip.yml
```

4. Edit the Ingress to use the port name:

```bash
vi my-ingress.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - path: /somepath
        pathType: Prefix
        backend:
          service:
            name: svc-clusterip
            port:
              name: http
```

```bash
kubectl apply -f my-ingress.yml
```

5. Check the status of the ingress again:

```bash
kubectl describe ingress my-ingress
```

---

## What is an Ingress

> An **Ingress** is a Kubernetes object that manages external access to Services in the cluster.
>
> An Ingress is capable of providing more functionality than a simple NodePort Service, such as SSL termination, advanced load balancing, or name-based virtual hosting.

## Ingress controllers

> Ingress objects actually do nothing by themselves. In order for Ingresses to do anything, you must intall one or more **Ingress controllers**.
>
> There are a variety of Ingress Controllers available - all of which implement different methods for providing external access to your Services.

## Routing to a Service

> Ingresses define a set of **routing rules**. A routing rule's properties determine to which requests it applies.
>
> Each rule has a set of **paths**, each with a **backend**. Requests matching a path will be routed to its associated backend.
>
> In the following example, a request to *http://<some-endpoint>/somepath* would be routed to port 80 on the my-service Service.
>
>```yaml
>apiVersion: networking.k8s.io/v1
>kind: Ingress
>metadata:
>  name: my-ingress
>spec:
>  rules:
>  - http:
>      paths:
>      - path: /somepath
>        pathType: Prefix
>        backend:
>          service:
>            name: my-service
>            port:
>              number: 80
>```
>

## Routing to a service with a Named Port

> If a Service uses a **named port**, and Ingress can also use the port's name to choose to which port it will route.
>
> ```yaml
>apiVersion: v1
>kind: Service
>metadata:
>  name: my-service
>spec:
>  selector:
>    app: MyApp
>  ports:
>    - name: web
>      protocol: web
>      port: 80
>```
>
>```yaml
>apiVersion: networking.k8s.io/v1
>kind: Ingress
>metadata:
>  name: my-ingress
>spec:
>  rules:
>  - http:
>      paths:
>      - path: /somepath
>        pathType: Prefix
>        backend:
>          service:
>            name: my-service
>            port:
>              name: web
>```
>

## Hands-On Demo
  
  *Note: See lesson reference.*
