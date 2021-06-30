# Using DaemonSets

> The Kubernetes scheduler allocates pods based on resource availability, but there are other ways to allocate pods to nodes. In this lesson, we look at DaemonSets. We will discuss what they are and how they differ from regular scheduling. We will also demonstrate how to create a DaemonSet.

## Relevant Docs

- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

---

## What Is a DaemonSet?

> **DaemonSet**
>  - Automatically runs a copy of a Pod on each node.
>  - DaemonSets will run a copy of the Pod on new nodes as they are added to the cluster

## DaemonSets and Scheduling

> DaemonSets respect normal scheduling rules around node labels, taints, and tolerations. If a pod would not normally be scheduled on a node, DaemonSet will not create a copy of the pod on that Node.

## Hands-On Demo

1. Create a yaml descriptor called `my-daemonset.yml`

  ```YAML
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: my-daemonset
  spec:
    selector:
      matchLabels:
        app: my-daemonset
    template:
      metadata:
        labels:
          app: my-daemonset
      spec:
        containers:
        - name: nginx
          image: nginx:1.19.1
  ```

2. Run `kubectl get daemonset`

  - Output:
    ```
    NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    my-daemonset   2         2         2       2            2           <none>          2m38s
    ```

3. Run `kubectl get pods -o wide`

  - Output:
    ```
    NAME                 READY   STATUS    RESTARTS   AGE     IP                NODE             NOMINATED NODE   READINESS GATES
    my-daemonset-ghjdt   1/1     Running   0          4m32s   192.168.247.27    instance-2-cka   <none>           <none>
    my-daemonset-rzkjb   1/1     Running   0          4m32s   192.168.174.101   instance-3-cka   <none>           <none>
    ```
  *Note: notice how each worker node is running the daemonset pods*