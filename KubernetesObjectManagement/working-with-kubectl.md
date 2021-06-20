# K8s Object Management

## Working with kubectl

> When working with Kubernetes, you are usually using kubectl, which is a powerful tool with many features that can help you navigate and manage your Kubernetes cluster. In this lesson, we will explore some features of kubectl — in particular, we'll focus on kubectl skills that might be helpful on the CKA exam.

## Relevant Docs:
  - [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)

---

## What is `kubectl`?

> `kubectl` is a command line tool that allows you to interact with k8s. kubectl uses the Kubernetes API to communicate with the cluster and carry out your commands

> "You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs."

## `kubectl get`

> Use `kubectl get` to list objects in the K8s cluster.

  - -o = Set output format
  - --sort-by = Sort output using a JSONPath expression
  - --selector = Filter results by label

```
kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>
```

## `kubectl describe`

> You can get detailed information about K8s objects using `kubectl describe`

```
kubectl describe <object type>/<object name>
```

## `kubectl create`

> Use `kubectl create` to create objects

> Supply a YAML files with `-f` to create an object from a YAML descriptor stored in the file

*Note: If you attempt to create an object that already exists, an error will occur*

```
kubectl create -f <file name>
```

## `kubectl apply`

> `kubectl apply` is similar to `kubectl create`. However if you use `kubectl apply` on an object that already exists, it will modify the existing object, if possible.

```
kubectl apply -f <file name>
```

*Note: used to update existing object, or create a new object*

## `kubectl delete`

> Use `kubectl delete` to delete objects from the cluster.

```
kubectl delete <object type>/<object name>
```

## `kubectl exec`

> `kubectl exec` can be used to run commands inside containers. Keep in mind that, in order for a command to succeed, the necessary software must exist within the container to run it.

```
kubectl exec <pod name> -c <container name> -- <command>
```

## Hands-On Demo

1. Create a pod:

  ```
  vi pod.yml
  ```

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
    - name: busybox
      image: radial/busyboxplus:curl
      command: ['sh', '-c', 'while ture; do sleep 3600; done']
  ```

2. Apply the YAML file:

  ```
  kubectl apply -f pod.yml
  ```

  - Output:
  ```
  pod/my-pod created
  ```

3. Use `kubectl get`

- You can enter a pod name or just get pods in order to display specific pods or list all, but here is a tip for viewing all available commands for `kubectl get`

  - Run:
    ```kubectl get api-resources```
    - Output:
      ```
      bgppeers                                       crd.projectcalico.org/v1               false        BGPPeer
      blockaffinities                                crd.projectcalico.org/v1               false        BlockAffinity
      clusterinformations                            crd.projectcalico.org/v1               false        ClusterInformation
      felixconfigurations                            crd.projectcalico.org/v1               false        FelixConfiguration
      globalnetworkpolicies                          crd.projectcalico.org/v1               false        GlobalNetworkPolicy
      globalnetworksets                              crd.projectcalico.org/v1               false        GlobalNetworkSet
      hostendpoints                                  crd.projectcalico.org/v1               false        HostEndpoint
      ipamblocks                                     crd.projectcalico.org/v1               false        IPAMBlock
      ipamconfigs                                    crd.projectcalico.org/v1               false        IPAMConfig
      ipamhandles                                    crd.projectcalico.org/v1               false        IPAMHandle
      ippools                                        crd.projectcalico.org/v1               false        IPPool
      kubecontrollersconfigurations                  crd.projectcalico.org/v1               false        KubeControllersConfiguration
      networkpolicies                                crd.projectcalico.org/v1               true         NetworkPolicy
      networksets                                    crd.projectcalico.org/v1               true         NetworkSet
      endpointslices                                 discovery.k8s.io/v1beta1               true         EndpointSlice
      events                            ev           events.k8s.io/v1                       true         Event
      ingresses                         ing          extensions/v1beta1                     true         Ingress
      flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta1   false        FlowSchema
      prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta1   false        PriorityLevelConfiguration
      ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
      ingresses                         ing          networking.k8s.io/v1                   true         Ingress
      networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy
      runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
      poddisruptionbudgets              pdb          policy/v1beta1                         true         PodDisruptionBudget
      podsecuritypolicies               psp          policy/v1beta1                         false        PodSecurityPolicy
      clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
      clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
      rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding
      roles                                          rbac.authorization.k8s.io/v1           true         Role
      priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
      csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
      csinodes                                       storage.k8s.io/v1                      false        CSINode
      storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
      volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
      ```

  - These are all available commands for `kubectl get`

4. Lets change the output format:

  - Run `kubectl get pods -o wide`
    ```
    NAME     READY   STATUS             RESTARTS   AGE     IP               NODE             NOMINATED NODE   READINESS GATES
    my-pod   0/1     CrashLoopBackOff   6          6m32s   192.168.174.72   instance-3-cka   <none>           <none>
    ```

  - Run `kubectl get pods -o json`
    ```JSON
                            "lastState": {
                            "terminated": {
                                "containerID": "containerd://977cd6db44d92cc23fded49ba276ed21f4ec30f6ad9bd4d4f2b484ea9eab2c23",
                                "exitCode": 0,
                                "finishedAt": "2021-05-24T21:51:58Z",
                                "reason": "Completed",
                                "startedAt": "2021-05-24T21:51:58Z"
                            }
                        },
                        "name": "busybox",
                        "ready": false,
                        "restartCount": 6,
                        "started": false,
                        "state": {
                            "waiting": {
                                "message": "back-off 5m0s restarting failed container=busybox pod=my-pod_default(dc278629-eba9-4ee6-9c4b-a64f4224ef67)",
                                "reason": "CrashLoopBackOff"
                            }
                        }
                    }
                ],
                "hostIP": "10.128.0.4",
                "phase": "Running",
                "podIP": "192.168.174.72",
                "podIPs": [
                    {
                        "ip": "192.168.174.72"
                    }
                ],
                "qosClass": "BestEffort",
                "startTime": "2021-05-24T21:46:20Z"
            }
        }
    ],
    "kind": "List",
    "metadata": {
        "resourceVersion": "",
        "selfLink": ""
    }
    ```

    *Note: above json is just a snippet, there is a lot more*

5. Run `kubectl get pods -o yaml`

  ```YAML
          type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2021-05-24T21:57:04Z"
      message: 'containers with unready status: [busybox]'
      reason: ContainersNotReady
      status: "False"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2021-05-24T21:46:20Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: containerd://c4b90663240b46a3f452f0f389f32ccd22ae94bac50e5e15065feb70c8cd7612
      image: docker.io/radial/busyboxplus:curl
      imageID: sha256:4776f1f7d1f625c8c5173a969fdc9ae6b62655a2746aba989784bb2b7edbfe9b
      lastState:
        terminated:
          containerID: containerd://c4b90663240b46a3f452f0f389f32ccd22ae94bac50e5e15065feb70c8cd7612
          exitCode: 0
          finishedAt: "2021-05-24T21:57:03Z"
          reason: Completed
          startedAt: "2021-05-24T21:57:03Z"
      name: busybox
      ready: false
      restartCount: 7
      started: false
      state:
        waiting:
          message: back-off 5m0s restarting failed container=busybox pod=my-pod_default(dc278629-eba9-4ee6-9c4b-a64f4224ef67)
          reason: CrashLoopBackOff
    hostIP: 10.128.0.4
    phase: Running
    podIP: 192.168.174.72
    podIPs:
    - ip: 192.168.174.72
    qosClass: BestEffort
    startTime: "2021-05-24T21:46:20Z"
  kind: List
  metadata:
    resourceVersion: ""
    selfLink: ""
  ```

6. Sort by JSONPath using `kubectl get pods -o wide --sort-by .spec.nodeName`

  ```
  NAME     READY   STATUS             RESTARTS   AGE   IP               NODE             NOMINATED NODE   READINESS GATES
  my-pod   0/1     CrashLoopBackOff   9          24m   192.168.174.72   instance-3-cka   <none>           <none>
  ```

7. Get all nodes in a certain namespace by using `kubectl get pods -n kube-system`

  ```
  NAME                                       READY   STATUS    RESTARTS   AGE
  calico-kube-controllers-6d7b4db76c-gcblx   1/1     Running   0          9d
  calico-node-6k7xv                          1/1     Running   0          21d
  calico-node-95xf8                          1/1     Running   0          21d
  calico-node-rr4fj                          1/1     Running   0          21d
  coredns-74ff55c5b-blgj7                    1/1     Running   0          9d
  coredns-74ff55c5b-xq2pp                    1/1     Running   0          9d
  etcd-instance-1-cka                        1/1     Running   0          21d
  kube-apiserver-instance-1-cka              1/1     Running   0          9d
  kube-controller-manager-instance-1-cka     1/1     Running   0          9d
  kube-proxy-8mvnf                           1/1     Running   0          9d
  kube-proxy-l9hzc                           1/1     Running   0          9d
  kube-proxy-s87gp                           1/1     Running   0          9d
  kube-scheduler-instance-1-cka              1/1     Running   0          9d
  ```

8. To filter by labels when using above command, use the `--selector` flag

  - `kubectl get pods -n kube-system --selector k8s-app=calico-node`

  ```
  NAME                READY   STATUS    RESTARTS   AGE
  calico-node-6k7xv   1/1     Running   0          21d
  calico-node-95xf8   1/1     Running   0          21d
  calico-node-rr4fj   1/1     Running   0          21d
  ```

9. Use command `kubectl describe pod my-pod` for detailed info about a certain pod

  ```
      Container ID:  containerd://2efbe5c75942def8f02fc4b534eff71c9e1d2766ebab307f900536ac120bff5e
    Image:         radial/busyboxplus:curl
    Image ID:      sha256:4776f1f7d1f625c8c5173a969fdc9ae6b62655a2746aba989784bb2b7edbfe9b
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while ture; do sleep 3600; done
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 25 May 2021 20:12:29 +0000
      Finished:     Tue, 25 May 2021 20:12:29 +0000
    Ready:          False
    Restart Count:  268
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ndb96 (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             False
      ContainersReady   False
      PodScheduled      True
    Volumes:
      default-token-ndb96:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-ndb96
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
      Type     Reason   Age                   From     Message
      ----     ------   ----                  ----     -------
      Normal   Pulled   41m (x261 over 22h)   kubelet  Container image "radial/busyboxplus:curl" already present on machine
      Warning  BackOff  72s (x6222 over 22h)  kubelet  Back-off restarting failed container
  ```

10. To create a pod use `kubectl create -f pod.yml`

  ```
  Error from server (AlreadyExists): error when creating "pod.yml": pods "my-pod" already exists
  ```

  *Note: this error occurs because the pod is already created*

11. To change your currently running pod, you can use `kubectl apply -f pod.yml`

  ```
  pod/my-pod unchanged
  ```
  *Note: this is unchanged because I did not change the yml file*

12. `kubectl exec my-pod -- echo "Hello, world!"` This is to run commands on a specific pod. Useful for diagnosis or installing dependancies.

  - If there are multiple containers, specifiy the container with `-c` after `my-pod`

  ```
  Hello, world!
  ```

*Note: this is displayed on the control plane, but actually takes place on the pod you specified*

13. Run `kubectl delete pod my-pod` in order to delete a specified pod

  ```
  pod "my-pod" deleted
  ```


