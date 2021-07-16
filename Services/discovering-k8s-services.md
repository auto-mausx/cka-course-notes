# Discovering K8s Services with DNS

> The Kubernetes DNS server does not just allow pods to locate one another directly, it also allows them to locate services. In this lesson, we will explore the relationship between Services and the Kubernetes DNS. We will also demonstrate how to interact with a service from within a pod using DNS.

## Relevant Docs

- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

## Lesson Reference

*Note: This lesson depends on objects that were created in the previous lesson. If you are following along, you will need to go through the previous lesson first.*

1. Get the IP address of the ClusterIP Service:

```bash
kubectl get service svc-clusterip
```

2. Perform a DNS lookup on the service from the busybox Pod:

```bash
kubectl exec pod-svc-test -- nslookup
<SERVICE_IP_ADDRESS>
```

3. Use the Service's IP address to make a request to the Service from the busybox pod:

```bash
kubectl exec pod-svc-test -- curl <SERVICE_IP_ADDRESS>
```

4. Make a request using the service name:

```bash
kubectl exec pod-svc-test -- curl svc-clusterip
```

5. Make a request using the service fully qualified domain name:

```bash
kubectl exec pod-svc-test -- curl svc-clusterip.default.svc.cluster.local
```

6. Create another busybox Pod in a new namespace:

```bash
kubectl create namespace new-namespace
```

```bash
vi pod-svc-test-new-namespace.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-svc-test-new-namespace
  namespace: new-namespace
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 10; done']
```

```bash
kubectl create -f pod-svc-test-new-namespace.yml
```

7. Attempt to make a request to the Service from the busybox Pod that is in another Namespace:

```bash
kubectl exec -n new-namespace pod-svc-test-new-namespace -- curl svc-clusterip
```

```bash
kubectl exec -n new-namespace pod-svc-test-new-namespace -- curl svc-clusterip.default.svc.cluster.local
```

8. The request using just the Service name will fail, but it will succeed when using the fully qualified domain name.

---

## Service DNS names

> The Kubernetes DNS (Domain Name Service) assigns **DNS names** to Services, allowing applications within the cluster to easily locate them.
>
> A service's fully qualified domain name has the following format:
>
> ```bash
> service-name.namespace-name.svc.cluster-domain.example
> ```
>
> The default cluster domain is `cluster.local`.

## Service DNS and Namespaces

> A service's fully qualified domain name can be used to reach the service from within **any Namespace** in the cluster.
>
> `my-service-my-namespace.svc.cluster.local`
>
> However, Pods within the same Namespace can also simply use the service name.
>
> `my-service`

## Hands-On Demo

  *Note: see lesson reference.*
