# Exploring K8s Persistent Volumes

> Persistent Volumes are a somewhat more advanced way to handle storage in Kubernetes. In this lesson, we discuss Persistent Volumes and how they differ from regular volumes. We also examine a few key Persistent Volume properties you should know.

## Relevant Docs

- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

---

## PersistentVolumes

> **PersistentVolumes** are K8s objects that allow you to treat storage as an abstract resource to be consumed by Pods, much like K8s treats compute resources such as memory and CPU.
>
> A PersistentVolume uses a set of attributes to describe the underlying storage resource (such as a disk or cloud storage location) which will be used to store data.
>
>```yaml
>kind: PersistentVolume
>apiVersion: v1
>metadata:
>  name: my-pv
>spec:
>  storageClassName: localdisk
>  capacity:
>    storage: 1Gi
>  accessModes:
>    - ReadWriteOnce
>  hostPath:
>    path: /var/output
>```

## Storage Classes

> Storage Classes allow k8s admins to specify the types of storage services they offer on their platform.
>
>```yaml
>apiVersion: storage.k8s.io/v1
>kind: StorageClass
>metadata:
>  name: localdisk
>provisioner: kubernetes.io/no-provisioner
>```
>
> For example, an admin could create a StorageClass called **slow** to describe low-performance but inexpensive storage resources, and another called **fast** for high-perfomance but more costly resources.
>
>```yaml
>apiVersion: storage.k8s.io/v1
>kind: StorageClass
>metadata:
>  name: slow
>provisioner: kubernetes.io/no-provisioner
>```
>
>```yaml
>apiVersion: storage.k8s.io/v1
>kind: StorageClass
>metadata:
>  name: fast
>provisioner: kubernetes.io/no-provisioner
>```
>
> This would allow users to choose storage resources that fit the needs of their applications.
>

## allowVolumeExpansion

> The **allowVolumeExpansion** property of a StorageClass determines whether or not the StorageClass supports the ability to resize volumes after they are created.
>
>```yaml
>apiVersion: storage.k8s.io/v1
>kind: StorageClass
>metadata:
>  name: fast
>provisioner: kubernetes.io/no-provisioner
>allowVolumeExpansion: true
>```
>
> If this property is not set to true, attempting to resize a volume that uses this StorageClass will result in an error.
>

## reclaimPolicies

> A PersistentVolume's **persistentVolumeReclaimPolicy** determines how the storage resources  can be reused when the PersistentVolume's associated PVC's are deleted.
>
>```yaml
>kind: PersistentVolume
>apiVersion: v1
>metadata:
>  name: my-pv
>spec:
>  storageClassName: localdisk
>  persistentVolumeReclaimPolicy: Recycle
>  capacity:
>    storage: 1Gi
>  accessModes:
>    - ReadWriteOnce
>  hostPath:
>    path: /var/output
>```
>
> - **Retain**: Keeps all data. This requires an admin to manually clean up the data and prepare the storage resource for reuse.
>
> - **Delete**: Deletes the underlying storage resource autmoatically (only works for cloud storage resources).
>
> - **Recycle**: Autmatically deletes all data in the underlying storage resource, allowing the PersistentVolume to be reused.
>

## PersistentVolumeClaims

> A **PersistentVolumeClaim** represents a user's request for storage resources. It defines a set of attributes similar to those of a PersistentVolume (StorageClass, etc.).
>
>```yaml
>apiVersion: v1
>kind: PersistentVolumeClaim
>metadata:
>  name: my-pvc
>spec:
>  storageClassName: localdisk
>  accessModes:
>    - ReadWriteOnce
>  resources:
>    requests:
>      storage: 100Mi
>```
> 
> When a PVC is created, it will look for a PersistentVolume that is able to meet the requested criteria. If it finds one, it will automatically be **bound** to the PersistentVolume.
>

## Using a PersistentVolumeClaim in a Pod

> PVC's can be **mounted** to a Pod's containers just like any other volume.
>
> If the PVC is bound to a PV, the containers will use the underlying PV storage.
>
>```yaml
>apiVersion: v1
>kind: Pod
>metadata:
>  name: pv-pod
>spec:
>  containers:
>  - name: busybox
>    image: busybox
>    volumeMounts:
>    - name: pv-storage
>      mountPath: /output
>  volumes:
>  - name: pv-storage
>    persistentVolumeClaim:
>      claimName: my-pvc
>```
>

## Resizing a PersistentVolumeClaim

> You can **expand** PVC's without interrupting applications that are using them.
>
> Simply edit the **spec.resources.requests.storage** attribute of an existing PVC, increasing its value.
>
> However, the StorageClass must support resizing volumes and must have **allowVolumeExpansion** set to **true**.
>
