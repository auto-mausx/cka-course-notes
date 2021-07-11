# Using NetworkPolicies

> The Kubernetes cluster network makes it very easy to establish network communication between Pods. However, this can present a challenge when it comes to securing your cluster. NetworkPolicies allow you to control network traffic into and out of your Pods. They are a great way to make your Kubernetes applications and infrastructure more secure. In this lesson, we will discuss NetworkPolicies and demonstrate how to create and use them.

## Relevant Docs

- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

## Lesson Reference

1. Create a new namespace.

```
kubectl create namespace np-test
```

2. Add a label to the Namespace.

```
kubectl label namespace np-test team=np-test
```

3. Create a web server Pod.

```
vi np-nginx.yml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: np-nginx
  namespace: np-test
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```
```
kubectl create -f np-nginx.yml
```

4. Create a client Pod.

```
vi np-busybox.yml
```
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: np-busybox
  namespace: np-test
  labels:
    app: client
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 5; done']
```
```
kubectl create -f np-busybox.yml
```

5. Get the IP address of the nginx Pod and save it to an environment variable.

```
kubectl get pods -n np-test -o wide

NGINX_IP=<np-nginx Pod IP>
```

6. Attempt to access the nginx Pod from the client Pod. This should succeed since no NetworkPolicies select the client Pod.

```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```

7. Create a NetworkPolicy that selects the Nginx Pod.

```
vi my-networkpolicy.yml
```
```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-networkpolicy
  namespace: np-test
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
```
```
kubectl create -f my-networkpolicy.yml
```
8. This NetworkPolicy will block all traffic to and from the Nginx Pod. Attempt to communicate with the Pod again. It should fail this time.

```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```

9. Modify the NetworkPolicy so that it allows incoming traffic on port 80 for all Pods in the np-test Namespace.

```
kubectl edit networkpolicy -n np-test my-networkpolicy
```
```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-networkpolicy
  namespace: np-test
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: np-test
    ports:
    - port: 80
      protocol: TCP
```

10. Attempt to communicate with the Pod again. This time, it should work!

```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```

---

## What Is a NetworkPolicy?

> A K8s `NetworkPolicy` is an object that allows you to control the flow of network communication to and from Pods.

> This allows you to build a more secure cluster network by keeping Pods isolated from traffic they do not need.

## Pod Selector

>  - **PodSelector**: Determines to which pods in the namespace the NetworkPolicy applies.

  - The podSelector can select Pods using Pod labels.

  ```YAML
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: my-network-policy
  spec:
    podSelector:
      matchLabels:
        role: db
  ```

  *Note: By default, Pods are considered non-isolated and completely open to all communication.*

  *If any NetworkPolicy selects a Pod, the Pod is considered isolated and will only be open to traffic by NetworkPolicies.*

## Ingress and Egress

* A NetworkPolicy can apply to Ingress, Egress, or both.

> **Ingress**: Incoming network traffic ocming into the Pod from another source.

### VS

> **Egress**: Outgoing network traffic leaving the Pod for another destination.

## `from` and `to` selectors

> **`from` selector**: Selects ingress (incoming) traffic that will be allowed.

> **`to` selector**: Selects egress (outgoing) traffic that will be allowed.

  - Example:
    ```YAML
    spec:
      ingress:
        - from:
          ...
      egress:
        - to:
          ...
    ```


### Types of `from` and `to` selectors

- **podSelector**: Select Pods to allow traffic from/to

  - Example:
    ```YAML
    spec:
      ingress:
        - from:
          - podSelector:
              matchLabels:
                app: db
    ```

- **namespaceSelector**: Select namespaces to allow traffic from/to.

  - Example:
    ```YAML
    spec:
      ingress:
        - from:
          - namespaceSelector:
              matchLabels:
                app: db
    ```

- **ipBlock**: Select an IP range to allow traffic from/to.

  - Example:
    ```YAML
    spec:
      ingress:
        - from:
          - ipBlock:
              cidr: 172.17.0.0/16
    ```

## Ports

> **`port`**: Specifies one or more ports that will allow traffic

  - Example
    ```YAML
    spec:
      ingress:
        - from:
          ports:
            - protocol: TCP
              port: 80
    ```

    *Note: Traffic is **ONLY** allowed if it matches both an allowed port **and** one of the from/to rules*

## Hands-On Demo

  *Note: see lesson reference*