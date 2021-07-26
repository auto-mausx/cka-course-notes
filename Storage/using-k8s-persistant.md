# Using K8s Persistent Volumes

> Kubernetes Persistent Volumes are a powerful way to manage storage in a cluster. In this lesson, we demonstrate how to create a Persistent Volume and mount storage to a container using a Persistent Volume Claim. We will also demonstrate how to expand a Persistent Volume Claim and how to recycle a Persistent Volume.

## Relevant Docs

- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

## Lesson Reference

1. Create a StorageClass that supports volume expansion:

```bash
vi localdisk-sc.yml
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: localdisk
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
```

```bash
kubectl create -f localdisk-sc.yml
```

2. Create a PersistentVolume:

```bash
vi my-pv.yml
```

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: my-pv
spec:
  storageClassName: localdisk
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/output
```

```bash
kubectl create -f my-pv.yml
```

3. Check the status of the PersistentVolume:

```bash
kubectl get pv
```

4. Create a PersistentVolumeClaim that will bind to the PersistentVolume:

```bash
vi my-pvc.yml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: localdisk
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

```bash
kubectl create -f my-pvc.yml
```

5. Check the status of the PersistentVolume and PersistentVolumeClaim to verify they have been bound:

```bash
kubectl get pv
```

```bash
kubectl get pvc
```

6. Create a Pod that uses the PersistentVolumeClaim:

```bash
vi pv-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo Success! > /output/success.txt']
    volumeMounts:
    - name: pv-storage
      mountPath: /output
  volumes:
  - name: pv-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

```bash
kubectl create -f pv-pod.yml
```

7. Expand the PersistentVolumeClaim and record the process:

```bash
kubectl edit pvc my-pvc --record
```

```yaml
...

spec:

  ...

  resources:
    requests:
      storage: 200Mi

```

8. Delete the Pod and the PersistentVolumeClaim:

```bash
kubectl delete pod pv-pod
```

```bash
kubectl delete pvc my-pvc
```

9. Check the status of the PersistentVolume to verify it has been successfully recycled and is available again:

```bash
kubectl get pv
```
