# Using K8s Services

> This lesson examines what it looks like to use Kubernetes services in practice. We will discuss the various service types, as well as demonstrate how to create a service in a cluster.

## Relevant Docs

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

## Lesson Reference

1. Create a deployment:

```BASH
vi deployment-svc-example.yml
```

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-svc-example
spec:
  replicas: 3
  selector:
    matchLabels:
      app: svc-example
  template:
    metadata:
      labels:
        app: svc-example
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```

```BASH
kubectl create -f deployment-svc-example.yml
```

2. Create a ClusterIP Service to expose the deployment's Pods within the cluster network:

```BASH
vi svc-clusterip.yml
```

```YAML
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  type: ClusterIP
  selector:
    app: svc-example
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

```BASH
kubectl create -f svc-clusterip.yml
```

3. Get a list of the Service's endpoints:

```BASH
kubectl get endpoints svc-clusterip
```

4. Create a busybox Pod to test your service:

```YAML
vi pod-svc-test.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-svc-test
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 10; done']
```

```BASH
kubectl create -f pod-svc-test.yml
```

5. Run a command within the busybox Pod to make a request to the service:

```BASH
kubectl exec pod-svc-test -- curl svc-clusterip:80
```

6. You should see the Nginx welcome page, which is being served by one of the backend Pods created earlier using a Deployment.

7. Create a NodePort Service to expose the Pods externally:

```BASH
vi svc-nodeport.yml
```

```YAML
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
spec:
  type: NodePort
  selector:
    app: svc-example
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

```bash
kubectl create -f svc-nodeport.yml
```

8. Test the service by making a request from your browser to `http://<CONTROL_PLANE_NODE_PUBLIC_IP>:30080`. You should see the Nginx welcome page.

---

## Service Types

> Each Service has a type. The **Service type** determines how and where the Service will expose your application. There are four service types:
>
> - ClusterIP
> - NodePort
> - LoadBalancer
> - ExternalName (outside the scope of CKA)

## ClusterIP Services

> **ClusterIP** Services expose applications **inside** the cluster network. Use them when your clients will be other Pods within the cluster.

## NodePort Services

> **NodePort** Services expose applications **outside** the cluster network. Use NodePort when applications or users will be accessing your application from outside the cluster.

## LoadBalancer Services

> **LoadBalancer** Services also expose applications **outside** the cluster network, but they use an **external cloud load balancer** to do so. This service type only works with cloud platforms that include load balancing functionality.

## Hands-On Demo

*Note: see lesson reference.*
