# Exploring K8s Scheduling

> In this section, we will take a closer look at various methods whereby Kubernetes determines where and when to run a pod’s containers. This video discusses the most common scenario: scheduling. We will briefly introduce the Kubernetes scheduler and explore how it works.

## Relevant Docs

- [Kubernetes Scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
- [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

---

## What is scheduling?

- Scheduling:

  The Process of assigning Pods to Nodes so kubelets can run them

- Scheduler:

  Control plane component that handles scheduling

## Scheduling Process

> The K8s scheduler selects a suitable Node for each Pod. It takes into account things like:
>  - Resource requests vs. available node resources
>  - Various configurations that affect scheduling using node labels

## nodeSelector

> You can configure a `nodeSelector` for your Pods to limit which Node(s) the Pod can be scheduled on.

> Node selectors use node labels to filter suitable nodes.

- Example:
  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
    nodeSelector:
      mylabel: myvalue
  ```

## nodeName

> You can bypass scheduling and assign a Pod to a specific Node by name using `nodeName`.

- Example:
  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
    nodeName: k8s-worker1
  ```

## Hands-On Demo

1. Check your nodes with `kubectl get nodes`

  - Output:
    ```
    NAME             STATUS   ROLES                  AGE   VERSION
    instance-1-cka   Ready    control-plane,master   56d   v1.20.2
    instance-2-cka   Ready    <none>                 56d   v1.20.2
    instance-3-cka   Ready    <none>                 56d   v1.20.2
    ```

2. Attach a label to one of the worker nodes

  - Run `kubectl label nodes instance-2-cka special=true`

    - Output:
      ```
      node/instance-2-cka labeled
      ```

3. Create a Pod called `nodeselector-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: nodeselector-pod
  spec:
    nodeSelector:
      special: "true"     <= Must be in double quotes
    containers:
    - name: nginx
      image: nginx:1.19.1
  ```

4. Run `kubectl get pod nodeselector-pod -o wide`

  - Output:
    ```
    NAME               READY   STATUS    RESTARTS   AGE   IP               NODE             NOMINATED NODE   READINESS GATES
    nodeselector-pod   1/1     Running   0          57s   192.168.247.26   instance-2-cka   <none>           <none>
    ```
  *Note: notice how the pod is running on the selected pod*


5. Create a pod def file called `nodename-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: nodeselector-pod
  spec:
    nodeName: instance-3-cka
    containers:
    - name: nginx
      image: nginx:1.19.1
  ```

6. Notice that the pod was spun up on the specified node

  ```
  NAME           READY   STATUS    RESTARTS   AGE   IP                NODE             NOMINATED NODE   READINESS GATES
  nodename-pod   1/1     Running   0          16s   192.168.174.100   instance-3-cka   <none>           <none>
  ```