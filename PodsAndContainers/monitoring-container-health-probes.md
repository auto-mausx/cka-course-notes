# Monitoring Container Health with Probes

> Kubernetes comes with a useful set of features for constantly monitoring the health and state of your containers. With a little customization, you can use these features to create rather robust self-healing applications. In this lesson, we discuss probes, a set of Kubernetes features that allow you to customize how Kubernetes determines the current state of a container.

## Relevant Docs:

- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

- [Container Probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)

---

## Container Health

> K8s provides a number of features that allow you to build robust solutions, such as the ability to automatically restart unhealthy containers. To make the most of these features, k8s needs to be able to accurately determine the status of your applications. This means actively monitoring **container health**.

## Liveness Probes

> **Liveness probes** allow you to automatically determine whether or not a container application is in a healthy state.

> By default, K8s will only consider a container to be "down" if the container process stops.

> Liveness probes allow you to costomize this detection mechanism and make it more sophisticated.

## Startup Probes

> **Startup probes** are very similar to liveness probes. However, while liveness probes run constanly on a schedule, startup probes run at container startup and stop running once they succeed.

> They are used to determine when the application has successfully started up. Startup probes are especially useful for legacy applications that can have  long startup times.

## Readiness Probes

> **Readiness probes** are used to determine when a container is ready to accept requests. When you have a service backed by multiple container endpoints, user traffic will not be sent to a particular pod until its containers have all passed the readiness checks defined by their readiness probes.

> Use readiness probes to preven user traffic from being sent to pods that are still in the process of starting up.

## Hands-On Demo

1. Create a pod named `liveness-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: liveness-pod
  spec:
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'while true; do sleep 3600; done']
      livenessProbe:
        exec:
          command: ['echo', 'Hello, World!']
        initialDelaySeconds: 5
        periodSeconds: 5
  ```

  ```
  kubectl create -f liveness-pod.yml
  ```

  - Output:

    ```pod/liveness-pod created ```

  ```
  kubectl get pod liveness-pod
  ```

  - Output:

  ```
    NAME           READY   STATUS    RESTARTS   AGE
    liveness-pod   1/1     Running   0          2m8s
  ```

2. Create a second pod named `liveness-pod-http.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: liveness-pod-http
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.1
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
  ```

  - Output:

    ```pod/liveness-pod-http created```

  - Verify it was created

    ```kubectl get pod liveness-pod-http```

  - Output:
    ```
    NAME                READY   STATUS    RESTARTS   AGE
    liveness-pod-http   1/1     Running   0          94s
    ```

3. Create a startup probe named `startup-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: startup-pod
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.1
      startupProbe:
        httpGet:
          path: /
          port: 80
        failureThreshold: 30
        periodSeconds: 10
  ```

  ```
  kubectl create -f startup-pod.yml
  ```

  - Output:

  ```pod/startup-pod created```

  - Verify:

  ```kubectl get pod startup-pod```

  - Output:

  ```
  NAME          READY   STATUS    RESTARTS   AGE
  startup-pod   0/1     Running   0          68s
  ```

4. Create a readiness probe called `readiness-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: readiness-pod
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.1
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
  ```

  ```
  kubectl create -f readiness-pod.yml
  ```

  - Output:

  ```pod/readiness-pod created```

  - Verify:
  ```
  kubectl get pod readiness-pod
  ```

  - Output:
  ```
  NAME            READY   STATUS    RESTARTS   AGE
  readiness-pod   1/1     Running   0          81s
  ```

