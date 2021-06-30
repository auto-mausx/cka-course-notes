# Using Static Pods

> The Kubernetes scheduler and DaemonSets provide different ways for the Kubernetes control plane to allocate pods. However, individual kubelets can also allocate pods on their own nodes using static pods. In this lesson, we discuss static pods, their use cases, and how to create them.

## Relevant Docs

- [Create Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

---

## What Is a Static Pod?

> **Static Pod**
>  - A pod that is managed directly by the kubelet on a node, not by the K8s API server. They can run even if there is no K8s API server present.

> Kubelet automatically creates static Pods from YAML manifest files located in the manifest path on the node.

## Mirror Pods

> **Mirror Pods**
>  - Kubelet will create a `mirror Pod` for each static Pod. Mirror Pods allow you to see the status of the static Pod via the K8s API, but you cannot change or manage them via the API.

## Hands-On Demo

1. Create the static pod **ON A WORKER NODE** using `sudo vi /etc/kubernetes/manifests/my-static-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-static-pod
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.1
  ```
  *Note: kubelet will actually automatically create this pod since it scans that file, but use this command to create it right away: `sudo systemctl restart kubelet`*


2. **ON THE CONTROL PLANE NODE** run `kubectl get pods`

  - Output:
    ```
    NAME                           READY   STATUS    RESTARTS   AGE
    my-daemonset-ghjdt             1/1     Running   0          19m
    my-daemonset-rzkjb             1/1     Running   0          19m
    my-static-pod-instance-2-cka   1/1     Running   0          2m59s
    ```
    *Note: see how the static pod now exists on the 2nd worker node. This is a mirror pod though, not the actual pod*

3. Try to delete that pod on the control plane node.

  - `kubectl delete pod my-static-pod-instance-2-cka`

  - Output:
    ```
    pod "my-static-pod-instance-2-cka" deleted
    ```

4. Check the pods running again...

  - Output:
    ```
    NAME                           READY   STATUS    RESTARTS   AGE
    my-daemonset-ghjdt             1/1     Running   0          22m
    my-daemonset-rzkjb             1/1     Running   0          22m
    my-static-pod-instance-2-cka   1/1     Running   0          72s
    ```
  *Note: notice how the pod didn't get deleted, and the mirror pod was recreated*