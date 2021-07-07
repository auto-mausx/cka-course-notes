# Scaling Applications with Deployments

> One of the use cases for deployments is application scaling. In this lesson, we explore various ways to scale an application using deployments. We will also demonstrate our knowledge by scaling an application in a Kubernetes cluster.

## Relevant Docs

- [Scaling a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)

---

## What is Scaling?

> **Scaling** refers to deditcating more (or fewer) resources to an application in order to meet changing needs.

> K8s Deployments are very useful in **horizontal scaling**, which involves changing the number of containers running an application.

## Deployment Scaling

> **Deployment Scaling**
>  - The Deployment's **replicas** settings determines how many replicas are desired in its desired state. If the **replicas** number is changed, replica Pods will be created or deleted to satisfy the new number.

## How to Scale a Deployment

>  - Option 1:
>    - You can scale a deployment simply by changing the number of **replicas** in the YAML descriptor with `kubectl apply` or `kubectl edit`
>       - Example:
>         ```YAML
>         ...
>         spec:
>           replicas: 5
>          ...
>          ```

>  - Option 2:
>    - Or use the special `kubectl scale` command.
>      - Example:
>        ```
>        $ kubectl scale \
>        deployment.v1.apps/my-deployment \
>        --replicas=5
>        ```

## Hands-On Demo

1. Spin up the `my-deployments.yml` deployment from last time.

2. Change the deployment replicas to `5` instead of `3` and save the file.

  ```YAML
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-deployment
  spec:
    replicas: 3 => 5
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

3. Apply the newly edited file with `kubectl -f apply my-deployment.yml`

4. Run `kubectl get deployments`
  - Output:
    ```
    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    my-deployment   5/5     5            5           22m
    ```
    *Note: notice how the replicas scaled up to 5 from the original 3*

5. Run `kubectl edit deployment my-deployment`

    - This will output a full description of the deployment where you can edit the replica count under the `spec:` field. **MUST BE CHANGED IN SPEC, NOT STATUS**

      *Note: as soon as the file is saved, it will change the replica amount*

6. Run `kubectl scale deployment.v1.apps/my-deployment --replicas=3`

  - Output:
    ```
    deployment.apps/my-deployment scaled
    ```

7. Run `kubectl get deployments` to see the new replica amount

  - Output:
    ```
    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    my-deployment   3/3     3            3           30m
    ```

