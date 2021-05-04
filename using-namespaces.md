# What Is a Namespace?


> Namespaces are virtual clusters backed by the same physical cluster. K8s objects, such as pods and containers, live in namespaces. Namespaces are a way to separate and organize objects in your cluster.


# List Existing Namespaces

You can list existing namespaces with `kubectl`:

```
$ kubectl get namespaces
```

All clusters have a default namespace. This is used when no other namespace is specified. kubeadm also creates a kube-system namespace for system components.

# Specify a Namespace

When using `kubectl`, you may need to specify a namespace. You can do this with the `--namespace` flag

```
$ kubectl get pods --namespace my-namespace
```

# Create a Namespace

You can create your own namespace with `kubectl`.

```
$ kubectl create namespace my-namespace
```


## Relevant Docs:

 - [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

# Hands-on Demo steps:

1. run `kubectl get namespaces`

    - Output:

    ```
    NAME              STATUS   AGE
    default           Active   45m
    kube-node-lease   Active   45m
    kube-public       Active   45m
    kube-system       Active   45m
    ```

   - Default is the default namespace

   - the kube-system namespace is contains system compoents that are part of the cluster itself.

2. run `kubectl get pods`
    - Output:

    ```
    No resources found in default namespace.
    ```

    - This is because it assumes the default namespace, and there are no pods there

3. run `kubectl get pods --namespace kube-system`

    - Output:

    ```
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-6d7b4db76c-8k2xm   1/1     Running   0          40m
    calico-node-6k7xv                          1/1     Running   0          40m
    calico-node-95xf8                          1/1     Running   0          22m
    calico-node-rr4fj                          1/1     Running   0          28m
    coredns-74ff55c5b-jwd4m                    1/1     Running   0          52m
    coredns-74ff55c5b-lfvs4                    1/1     Running   0          52m
    etcd-instance-1-cka                        1/1     Running   0          53m
    kube-apiserver-instance-1-cka              1/1     Running   0          53m
    kube-controller-manager-instance-1-cka     1/1     Running   0          53m
    kube-proxy-dszrp                           1/1     Running   0          22m
    kube-proxy-m8hf5                           1/1     Running   0          28m
    kube-proxy-n8kph                           1/1     Running   0          52m
    kube-scheduler-instance-1-cka              1/1     Running   0          53m
    ```

    - Be sure to specifiy which namespace you would like to search for pods in

    - You can run `kubectl get pods --all-namespaces` to search all available namespaces for pods

4. run `kubectl create namespace my-namespace`

    - Output:

    ```
    namespace/my-namespace created
    ```
    - run `kubectl get namespaces` to see newly created namespace
    ```
    NAME              STATUS   AGE
    default           Active   58m
    kube-node-lease   Active   58m
    kube-public       Active   58m
    kube-system       Active   58m
    my-namespace      Active   32s  <-- new
    ```

