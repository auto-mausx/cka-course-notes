# Managing Rolling Updates with Deployments

> Deployments are a great way to manage rolling updates and rollbacks for Kubernetes applications. In this lesson, we examine the process of performing rolling updates and rollbacks using deployments. This will give you some insight into how deployments can be used to manage changes to your applications.

## Relevant Docs

- [Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

- [Rolling Back a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)

---

## What is a Rolling Update?

> **Rolling Update**
>  - **Rolling updates** allow you to make changes to a Deployment's Pods at a controlled rate, gradually replacing old Pods with new Pods. This allows you to update your Pods without incurring downtime.

## What is a Rollback?

> **Rollback**
>  - If an update to a deployment causes a problem, you can **roll back** the deployment to a previous working state.

## Hands-On Demo


1. Edit an exsisting deployment with `kubectl edit deployment my-deployment` and change the container image to `nginx:1.19.2`

  ```YAML
  ---
  ...
  template:
      metadata:
        creationTimestamp: null
        labels:
          app: my-deployment
      spec:
        containers:
        - image: nginx:1.19.1  <== nginx:1.19.2
          imagePullPolicy: IfNotPresent
          name: nginx
          ports:
          - containerPort: 80
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  ...
  ---
  ```

2. Run `kubectl rollout status deployment.v1.apps/my-deployment`

  - Output:
    ```
    deployment "my-deployment" successfully rolled out
    ```

  *Note: Notice how the deployment is **ALREADY** rolled out, hence checking the status. This happens immediately upon editing the deployment*

3. Another way to change the deployment is to run `kubectl set image deployment/my-deployment nginx=nginx:broken --record`

  - *Note: `nginx` is the container name. Also use `--record` in order to make it easier to roll back and see what was changed*

4. Run `kubectl rollout status deployment.v1.apps/my-deployment` to see the status.

  - Output:

  ```
  Waiting for deployment "my-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
  ```

  ```
  NAME                             READY   STATUS             RESTARTS   AGE
  my-daemonset-9btwv               1/1     Running            1          6d21h
  my-daemonset-xdrbd               1/1     Running            1          6d21h
  my-deployment-75ff55584d-nmdwb   0/1     ImagePullBackOff   0          2m24s
  my-deployment-7d655f6776-4hltk   1/1     Running            0          8m53s
  my-deployment-7d655f6776-c99nf   1/1     Running            0          8m46s
  my-deployment-7d655f6776-wtqrw   1/1     Running            0          8m44s
  my-static-pod-instance-2-cka     1/1     Running            2          6d21h
  ```

  *Note: Notice how the old pods are still running since the new pod cannot pull the broken image*

5. Run `kubectl rollout history deployment.v1.apps/my-deployment`

  - Output:
    ```
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    3         kubectl set image deployment/my-deployment nginx=nginx:broken --record=true
    ```

6. Run `kubectl rollout undo deployment.v1.apps/my-deployment` to undo the last revision to the deployment

  *Note: In order to select a specific revision, use the `--to-revision` flag*