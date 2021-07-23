# K8s Storage Overview

> In this section, we will examine storage in the context of Kubernetes. This lesson introduces how Kubernetes handles storage. It will also provide a brief overview of what we will cover in this section.

## Relevant Docs

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

- [Persistant Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

---

## Container File Systems

> The container file system is **ephemeral**. Files on the container's file system exist only as long as the container exists.
>
> If a container is deleted or re-created in K8s, data stored on the container file system is lost.

## Volumes

> Many applications need a more persistant method of data storage.
>
>**Volumes** allow you to store data outside the container file system while allowing the container to access the data at runtime.
>
> This can allow data to persist beyond the life of the container.

## Persistant Volumes

> Volumes offer a simple way to provide external storage to containers within the Pod/container spec.
>
> **Persistent Volumes** are a slightly more advanced form of Volume. They allow you to treat storage as an abstract resource and consume it using your Pods.

## Volume Types

> Both Volumes and Persistant Volumes each have a **volume type**. The volume type determines how the storage is actually handled.
>
> Various volume types support storage methods such as:
>
> - NFS
> - Cloud storage mechanisms (AWS, Azure, GCP)
> - ConfigMaps and Secrets
