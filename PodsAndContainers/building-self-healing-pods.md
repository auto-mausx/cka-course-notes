# Building Self-Healing Pods with Restart Policies

> Kubernetes restart policies instruct the Kubernetes system on how to handle unhealthy pods. When combined with probes, restart policies can allow Kubernetes to automatically detect pod failures and take corrective action. In this lesson, we examine restart policies and look at how to use them.

## Relevant Docs

- [Restart Policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)

---

## Restart Policies

> K8s can automatically restart containers when they fail. **Restart policies** allow you to customize this behavior by defining when you want a pod's containers to be automatically restarted.

> Restart policies are an important component of self-healing applications, which are automatically repaired when a problem arises

> There are three possible values for a pod's restart policy in K8s: **Always**, **OnFailure**, and **Never**.

## Always

> **Always** is the default restart policy in K8s. With this policy, containers will always be restarted if they stop, even if they completed sucessfully. Use this policy for applications that should always be running.

## OnFailure

> The **OnFailure** restart policy will restart containers only if the container process exits with an error code or the container is determined to be unhealthy by a liveness probe. Use this policy for applications that need to be run successfully and then stop.

## Never

> The **Never** restart policy will cause the pod's containers to never be restarted, even if the container exits or a liveness probe fails. Use this for applications that should run once and never be automatically restarted.

## Hands-On Demo

1. Create an Always pod named `always-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: always-pod
  spec:
    restartPolicy: Always
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10']
  ```

  ```
  kubectl create -f always-pod.yml
  ```

  - Output:

    ```pod/always-pod created```

  - Verify:

    ```
    NAME         READY   STATUS    RESTARTS   AGE
    always-pod   1/1     Running   1          20s
    ```
    *Note: see number of restarts: since the pod is set to sleep every 10s, the pod will always restart*

2. Create a pod named `onfailure-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: onfailure-pod
  spec:
    restartPolicy: OnFailure
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10']
  ```

  ```
  kubectl create -f onfailure-pod.yml
  ```

  - Output:

  ```pod/onfailure-pod created```

  - Verify:

  ```
  NAME            READY   STATUS             RESTARTS   AGE
  always-pod      0/1     CrashLoopBackOff   5          6m19s
  onfailure-pod   0/1     Completed          0          69s
  ```

  *Note: Notice how the onfailure pod did not restart after it went to sleep since it did not fail.*

3. Delete the OnFailure pod and change the yaml file then restart pod, observe what happens

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: onfailure-pod
  spec:
    restartPolicy: OnFailure
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']
  ```

  - Result:

  ```
  $ kubectl get pods

  NAME            READY   STATUS             RESTARTS   AGE
  always-pod      0/1     CrashLoopBackOff   6          11m
  onfailure-pod   1/1     Running            0          10s

  $ kubectl get pods

  NAME            READY   STATUS             RESTARTS   AGE
  always-pod      0/1     CrashLoopBackOff   6          11m
  onfailure-pod   0/1     Error              0          12s

  $ kubectl get pods

  NAME            READY   STATUS             RESTARTS   AGE
  always-pod      0/1     CrashLoopBackOff   6          11m
  onfailure-pod   1/1     Running            1          14s
  ```

  *Note: notice on the second and last get pods, the pod errored, then restarted since it failed*

4. Create a pod called `never-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: never-pod
  spec:
    restartPolicy: Never
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']
  ```

  - Result:

  ```
  NAME        READY   STATUS   RESTARTS   AGE
  never-pod   0/1     Error    0          13s
  ```
  *Note: The pod has never restarted even though it was errored, hence never pod*