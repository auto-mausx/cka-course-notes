# Using K8s Volumes

> Volumes are a simple and easy way to mount external storage to your Kubernetes containers. In this lesson, we will examine volumes and the various options available when creating them. We will demonstrate how they can be used by mounting a simple volume to a container in our cluster.

## Relevant Docs

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

## Lesson Reference

1. Create a Pod that uses a hostPath volume to store data on the host:

```bash
vi volume-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo Success! > /output/success.txt']
    volumeMounts:
    - name: my-volume
      mountPath: /output
  volumes:
  - name: my-volume
    hostPath:
      path: /var/data
```

```bash
kubectl create -f volume-pod.yml
```

2. Check which worker node the pod is running on:

```bash
kubectl get pod volume-pod -o wide
```

3. Log in to that host and verify the contents of the output file:

```bash
cat /var/data/success.txt
```

4. Create a multi-container Pod with an emptyDir volume shared between containers:

```bash
vi shared-volume-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  containers:
  - name: busybox1
    image: busybox
    command: ['sh', '-c', 'while true; do echo Success! > /output/output.txt; sleep 5; done']
    volumeMounts:
    - name: my-volume
      mountPath: /output
  - name: busybox2
    image: busybox
    command: ['sh', '-c', 'while true; do cat /input/output.txt; sleep 5; done']
    volumeMounts:
    - name: my-volume
      mountPath: /input
  volumes:
  - name: my-volume
    emptyDir: {}
```

```bash
kubectl create -f shared-volume-pod.yml
```

5. Check the container log for busybox2. You should see the data that was generated by busybox1:

```bash
kubectl logs shared-volume-pod -c busybox2
```

---

## Volumes and volumeMounts

> Regular Volumes can be set up relatively easily within a Pod/container specification.
>
>**volumes**: In the Pod spec, these specify the storage volumes available to the Pod. They specify the volume type and other data that determines where and how the data is actually stored.
>
> **volumeMounts**: In the container spec, these reference the volumes in the Pod spec and provide a `mountPath` (the location on the file system where the container process will access the volume data).
>
> ```yaml
>apiVersion: v1
>kind: Pod
>metadata:
>  name: volume-pod
>spec:
>  containers:
>  - name: busybox
>    image: busybox
>    volumeMounts:
>    - name: my-volume
>      mountPath: /output
>  volumes:
>  - name: my-volume
>    hostPath:
>      path: /data

## Sharing Volumes between Containers

> You can use `volumeMounts` to mount the same volume to multiple containers within the same Pod.
>
> This is a powerful way to have multiple containers interact with one another. For example, you could create a secondary sidecar container that processes or transforms output from another container.
>
>```yaml
>apiVersion: v1
>kind: Pod
>metadata:
>  name: volume-pod
>spec:
>  containers:
>  - name: busybox
>    image: busybox
>    volumeMounts:
>    - name: my-volume
>      mountPath: /output
>  - name: busybox2
>    image: busybox
>    volumeMounts:
>    - name: my-volume
>      mountPath: /input
>  volumes:
>  - name: my-volume
>    emptyDir: {}

## Common Volume Types

> There are many volume types, but there are two you may want to especially be aware of.
>
> **hostPath**: Stores data in a specified directory on the K8s node.
>
> **emptyDir**: Stores data in a dynamically created location on the node. This directory exists only as long as the Pod exists on the node. The directory and the data are deleted when the Pod is removed. This volume type is very useful for simply sharing data between containers in the same Pod.

## Hands-On Demo

  *Note: see lesson reference.*
