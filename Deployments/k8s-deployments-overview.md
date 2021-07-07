# K8s Deployments Overview

> Deployments provide a useful set of features for automation in Kubernetes. In this lesson, we introduce deployments. We will also demonstrate the process of creating a basic deployment and examine the deployment specification.

## Relevant Docs

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---

## What is a Deployment?

>  **Deployment**
>  - A K8s object that defines a **desired state** for a ReplicaSet (a set of replica Pods). The Deployment Controller seeks to maintain the desired state by creating, deleting, and replacing Pods with new configurations.

## Desired State

> A Deployment's desired state includes these components:
>  - Replicas: The number of replica Pods the Deployment will seek to maintain
>  - Selector: A label selector used to identify the replica Pods managed by the deployment
>  - Template: A template Pod definition used to create replica Pods

## Use cases

> There are many use cases for Deployments, such as:
>  - Easily scale an application up or down by changing the number of replicas
>  - Perform rolling updated to deploy a new software version
>  - Roll back to a previous software version.

## Hands-On Demo

1. Create a file called `my-deployment.yml`

  ```YAML
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: my-deployment
    template:
      metadata:
        labels:
          app: my-deployment
      spec:
        containers:
        - name: nginx
          image: nginx:1.19.1
          ports:
          - containerPort: 80
  ```

  - Output:

    ```
    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    my-deployment   3/3     3            3           70s
    ```

2. If you delete one of the pods, it will spin up a new pod with a different name in order to maintain the replicaset of 3 replicas.