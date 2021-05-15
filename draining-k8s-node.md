# Safely Draining a K8s Node

> In order to perform maintenance on your Kubernetes nodes, you need to be able to transfer workloads to other nodes so maintenance tasks do not interfere with ongoing service. In this lesson, we walk through the process of draining a Kubernetes node. This will allow us to shift workloads to other nodes so maintenance can be performed.

### Relevant Docs:

- [Draining a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

---

## What is Draining?

- When performing maintenance, you may sometimes need to remove a K8s node from service.

- To do this, you can drain a node. Containers running on the node will be gracefully terminated (and potentially rescheduled on another node).

## Draining a Node

- To drain a node, use the `kubectl drain` command

```
$ kubectl drain <node name>
```

## Ignoring DaemonSets

- When draining a node, you may need to ignore DaemonSets (pods that are tied to each node). If you have any DeamonSet pods running on the node, you will likely need to use the `--ignore-daemonsets` flag

```
$ kubectl drain <node name> --ignore-daemonsets
```

## Uncordoning a Node

- If the node remains part of the cluster, you can allow pods to run on the node again when maintenance is complete using the `kubectl uncordon` command.

```
$ kubectl uncordon <node name>
```

## Hands-On Demonstration

1. create a `pod.yml` file on a control node with the following configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  restartPolicy: OnFailure
```

2. run `kubectl apply -f pod.yml` to apply the pod

    - Output:
    ```
    pod/my-pod created
    ```

3. Create a `deployment.yml` file also on the control node

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-deployment
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

4. run `kubectl apply -f deployment.yml` to create the deployment

    - Output:
    ```
    deployment.apps/my-deployment created
    ```

5. run `kubectl get pods -o wide` to view which node the pods are running on.

    - Output:
    ```
    NAME                             READY   STATUS    RESTARTS   AGE     IP               NODE             NOMINATED NODE   READINESS GATES
    my-deployment-5f85c44867-bzq64   1/1     Running   0          3m27s   192.168.247.1    instance-2-cka   <none>           <none>
    my-deployment-5f85c44867-c4hz5   1/1     Running   0          3m27s   192.168.247.2    instance-2-cka   <none>           <none>
    my-pod                           1/1     Running   0          9m43s   192.168.174.65   instance-3-cka   <none>           <none>
    ```

  - **Note:** *if the deployment is not running on different nodes, increase the number of instances until they do.*

6. Drain the node with the `my-pod` pod in it using `kubectl drain <instance-3-cka>`

    - Output:
    ```
    node/instance-3-cka cordoned
    error: unable to drain node "instance-3-cka", aborting command...
    There are pending nodes to be drained:
    instance-3-cka
    cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): default/my-
    pod
    cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/calico-node-95xf8, kube-system/kube-proxy-dszrp
    ```

- It cannot drain the node for 2 reasons:

    1.  Because you just created that single pod by itself, it's not able to automatically reschedule that pod to another worker node. It's going to abort the drain process since it doesn't want to just delete the pod.
    2. It is not able to delete the DaemonSet managed pods.

7. to get around this error, run `kubectl drain <instance-3-cka> --ignore-daemonsets --force`

    - Output:
    ```
    node/instance-3-cka already cordoned
    WARNING: deleting Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet: default/my-pod; ignoring DaemonSe
    t-managed Pods: kube-system/calico-node-95xf8, kube-system/kube-proxy-dszrp
    evicting pod default/my-pod
    evicting pod default/my-deployment-5f85c44867-pvcrh
    pod/my-pod evicted
    pod/my-deployment-5f85c44867-pvcrh evicted
    node/instance-3-cka evicted
    ```

8. run `kubectl get pods -o wide`
    - Output:
    ```
    NAME                             READY   STATUS    RESTARTS   AGE   IP              NODE             NOMINATED NODE
   READINESS GATES
    my-deployment-5f85c44867-4bcft   1/1     Running   0          9d    192.168.247.3   instance-2-cka   <none>
      <none>
    my-deployment-5f85c44867-bzq64   1/1     Running   0          10d   192.168.247.1   instance-2-cka   <none>
      <none>
    my-deployment-5f85c44867-c4hz5   1/1     Running   0          10d   192.168.247.2   instance-2-cka   <none>
      <none>
   ```

     - *Note: the deployment pods have been recreated and the "my-pod" pod has not, this is because it must stay true to the deployment yml file replicas (3). And they are all running on the same node since the other was drained. New pods were spun up to facilitate this condition*

9. run `kubectl get nodes`
    - Output:
    ```
    NAME             STATUS                     ROLES                  AGE   VERSION
    instance-1-cka   Ready                      control-plane,master   11d   v1.20.1
    instance-2-cka   Ready                      <none>                 11d   v1.20.1
    instance-3-cka   Ready,SchedulingDisabled   <none>                 11d   v1.20.1   <= Note the status
    ```

    - *Note: this means k8s will not run additional pods on this node since maintenance tasks are still on-going*

10. run `kubectl uncordon instance-3-cka`
    - Output:
    ```
    node/instance-3-cka uncordoned
    ```

    - *Note: This has now uncordoned the node so it can be used again*

11. run `kubectl get nodes`
    - Output:
    ```
    NAME             STATUS   ROLES                  AGE   VERSION
    instance-1-cka   Ready    control-plane,master   11d   v1.20.1
    instance-2-cka   Ready    <none>                 11d   v1.20.1
    instance-3-cka   Ready    <none>                 11d   v1.20.1  <= Note the status
    ```

    - *Note: the node is now up and running, ready to use again*

12. run `kubectl get pods -o wide`
    - Output:
    ```
    NAME                             READY   STATUS    RESTARTS   AGE   IP              NODE             NOMINATED NODE   READINESS GATES
    my-deployment-5f85c44867-4bcft   1/1     Running   0          9d    192.168.247.3   instance-2-cka   <none>           <none>
    my-deployment-5f85c44867-bzq64   1/1     Running   0          10d   192.168.247.1   instance-2-cka   <none>           <none>
    my-deployment-5f85c44867-c4hz5   1/1     Running   0          10d   192.168.247.2   instance-2-cka   <none>           <none>
    ```

    - *Note: Take note that uncordoning the node did not change the deployment back to the previously drained node, all the deployments are still on the same node as it was after the previous node was drained*

13. run `kubectl delete deployment my-deployment`
    - Output:
    ```
    deployment.apps "my-deployment" deleted
    ```

    - *Note: this is to clean up cluster for future labs*