# Understanding K8s DNS

> Kubernetes has a pluggable DNS setup, and our kubeadm-based cluster includes CoreDNS as a DNS solution. In this lesson, we explore how DNS functions within a Kubernetes cluster. We will also talk about how pods can use DNS to locate one another via the Kubernetes virtual network.

## Relevant Docs

- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

---

## DNS in K8s

> The K8s virtual network uses a **DNS** (Domain Name system) to allow Pods to locate other Pods and Services using domain names instead of IP addresses.

> This DNS runs as a Service within the cluster. You can usually find it in the `kube-system` namespace.

> Kubeadm clusters use **CoreDNS**

## Pod Domain names

> All pods in our kubeadm cluster are automatically given a domain name of the following form.

  ```
  pod-ip-address.namespace-name.pod.cluster.local
  ```

> A Pod in the `default` namespace with the IP address 192.168.10.100 would have a domain name like this:

  ```
  192-168-10-100.default.pod.cluster.local
  ```

## Hands-On Demo

1. Create pods called `dnsteset-pods.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: busybox-dnstest
  spec:
    containers:
    - name: busybox
      image: radial/busyboxplus:curl
      command: ['sh', '-c', 'while true; do sleep 3600; done']
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-dnstest
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.2
      ports:
      - containerPort: 80
  ```

  *Note: the `---` above lets you create multiple K8s objects in one YML file*

2. Run `kubectl get pods nginx-dnstest -o wide` in order to get the IP address

  - Output:
    ```
    NAME            READY   STATUS    RESTARTS   AGE   IP                NODE             NOMINATED NODE   READINESS GATES
    nginx-dnstest   1/1     Running   0          72s   192.168.174.122   instance-3-cka   <none>           <none>
    ```

3. Run a command inside one of the pods using `kubectl exec busybox-dnstest -- curl 192.168.174.122`

  - Output:
    ```
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
    100   612  100   612    0     0  91822      0 --:--:-- --:--:-- --:--:--   99k
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

    *Note: notice how we can communcate between pods with just an IP address*

4. Next we will do the same with a domain name: Run `kubectl get pods -n kube-system` to see the DNS service running

  - Output:
    ```
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-6d7b4db76c-gcblx   1/1     Running   10         56d
    calico-node-6k7xv                          1/1     Running   7          67d
    calico-node-95xf8                          1/1     Running   7          67d
    calico-node-rr4fj                          1/1     Running   7          67d
    coredns-74ff55c5b-blgj7                    1/1     Running   7          56d
    coredns-74ff55c5b-xq2pp                    1/1     Running   7          56d
    etcd-instance-1-cka                        1/1     Running   7          67d
    kube-apiserver-instance-1-cka              1/1     Running   10         56d
    kube-controller-manager-instance-1-cka     1/1     Running   8          56d
    kube-proxy-8mvnf                           1/1     Running   7          56d
    kube-proxy-l9hzc                           1/1     Running   7          56d
    kube-proxy-s87gp                           1/1     Running   7          56d
    kube-scheduler-instance-1-cka              1/1     Running   7          56d
    metrics-server-5556757555-mqgx7            1/1     Running   11         41d
    ```

5. Use the IP of the `nginx-dnstest` pod and do an `nslookup` of the IP like so: `kubectl exec busybox-dnstest -- nslookup 192-168-174-122.default.pod.cluster.local`

  - Output:
    ```
    This should have been the DNS name, but for some reason it didn't work :(
    ```

    *Note: moral of the story, you should be able to curl either the IP **OR** DNS name*


