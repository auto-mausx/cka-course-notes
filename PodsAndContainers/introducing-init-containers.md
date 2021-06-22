# Introducing Init Containers

> Init containers are a great way to run custom, complex tasks as part of the startup process for your main containers. In this lesson, we explore what init containers are and why you might want to use them. We will also demonstrate how to use init containers.

## Relevant Docs

- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

---

## Init Containers

> **Init containers** are containers that run once during the startup process of a pod. A pod can have any number of init containers, and they will each run once (in order) to completion.

  - Init Container 1

    - Init containers run to completion before the next init container or app containers start.

  - Init Container 2

    - You can have any number of init containers, and each will run in order.

  - App Containers

    - App containers will start up only once all init containers have completed.

> You can use init containers to perform a variety of starup tasks. They can contain and use software and setup scripts that are not needed by your main containers.

> They are often useful in keeping your main containers lighter and more secure by offloading startup tasks to a separate container.

## Use Cases for Init Containers

  **Some sample use cases for init containers:**

  - Cause a pod to wait for another K8s resource to be created before finishing startup.

  - Perform sensitive startup steps securely outside of app containers

  - Populate data into a shared volume at startup.

  - Communicate with another service at startup.

## Hands-On Demo

1. Create an init container called `init-pod`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: init-pod
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.1
    initContainers:
    - name: delay
      image: busybox
      command: ['sleep', '30']
  ```

  - Verify:
    ```
    NAME       READY   STATUS     RESTARTS   AGE
    init-pod   0/1     Init:0/1   0          3s
    ```
    *Note: Notice how the pod has not started yet, due to our app pod being slept (30 sec delay)*

2. Observe how the pod completes after 30 seconds

  ```
  NAME       READY   STATUS    RESTARTS   AGE
  init-pod   1/1     Running   0          2m19s
  ```