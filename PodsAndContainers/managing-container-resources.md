# Managing Container Resources

> Kubernetes is all about dynamically allocating resources. In this lesson, we examine some ways in which you can gain more control over the resources allocated to your pods. We will explore resource requests and limits, as well as how they affect pods and containers in Kubernetes.

## Relevant Docs:

- [Resource Requests and Limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container)

---

## Resource Requests

> **Resource Requests** allow you to define an amount of resources (such as CPU or memory) you expect a container to use. The K8s scheduler will use resource requests to avoid scheduling pods on nodes that do not have enough available resources.

*Note: Containers are allowed to use more (or less) than the requested resources. Resource requests only effect scheduling*

> Memory is meausred in bytes. CPU is measured in CPU units, which are 1/1000 of one CPU.

  - Example:

    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
    spec:
      containers:
      - name: busybox
        image: busybox
        resources:
          requests:
            cpu: "250m"
            memory: "128Mi"
    ```
    *Note: part of container spec*

## Resource Limits

> **Resource limits** provide a way for you to limit the amount of resources your containers can use. The container runtime is responsible for enforcing these limits, and different container runtimes do this differently.

*Note: Some runtimes will enforce these limits by terminating container processes that attempt to use more than the allowed amount of resources*

  - Example:
    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
    spec:
      containers:
      - name: busybox
        image: busybox
        resources:
          limits:
            cpu: "250m"
            memory: "128Mi"
    ```

## Hands-On Demo

1. Create a pod `big-request-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: big-request-pod
  spec:
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'while true; do sleep 3600; done']
      resources:
        requests:
          cpu: "10000m"
          memory: "128Mi"
  ```

  ``` kubectl create -f big-request-pod.yml ```

  - Output:
    ```
    pod/big-request-pod created
    ```

2. Check the big request pod

  ``` kubectl get pod big-request-pod ```

  - Output:
    ```
    NAME              READY   STATUS    RESTARTS   AGE
    big-request-pod   0/1     Pending   0          106s
    ```

*Note: the K8s scheduler is waiting for us to give it a worker node that can handle a 10 CPU request. If a worker node that can handle that request is never created, then this request will never run and will stay pending*

3. Create another pod called `resource-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: resource-pod
  spec:
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'while true; do sleep 3600; done']
      resources:
        requests:
          cpu: "250m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
  ```

  *Note: Docker will throttle processes based on CPU limit. It will also kill the process if the memory limit is exceeded.*

  ``` kubectl create -f resource-pod.yml ```

  - Output:
    ``` pod/resource-pod created ```

4. Use `kubectl get pods` to see which pods were created

  - Output:
  ```
  NAME              READY   STATUS             RESTARTS   AGE
  big-request-pod   0/1     Pending            0          10m
  env-pod           0/1     CrashLoopBackOff   14         49m
  my-pod            1/1     Running            1          21d
  resource-pod      1/1     Running            0          56s
  volume-pod        1/1     Running            0          38m
  ```

*Note: notice how it didn't make the giga request pod since we don't have a worker node capable of supporting a 10 CPU request*