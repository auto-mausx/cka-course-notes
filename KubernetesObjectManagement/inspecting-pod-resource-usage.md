# Inspecting Pod Resources Usage

> When you are running applications on Kubernetes, it may be necessary to take a closer look at the resources your pods are using. In this lesson, we will explore how you can investigate resource usage at the pod level. This will allow you to gain some insight into just how your compute resources are being used.

## Relevant Docs:

- [Tools for Monitoring Resources](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/
- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods)
- [Lesson Reference](https://acloudguru-content-attachment-production.s3-accelerate.amazonaws.com/1596636260278-devops-wb002%20-%20S04-L04%20Inspecting%20Pod%20Resource%20Usage.pdf)

---

## Kubernetes Metrics Server

  > In order to view metrics about the resources pods and containers are using, we need an add-on to collect and provide that data. One such add-on is **Kubernetes Metrics Server**.

## `kubectl top`

  > with `kubectl top`, you can view data about resource usage in your pods and nodes. `kubectl top` also supports flags like `--sort-by` and `--selector`

  ```
  $ kubectl top pod --sort-by <JSONPATH> --selector <selector>
  ```

## Hands-On Demo

1. Install K8s Metrics server.

  ```kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml```

    - Output:
      ```
      clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
      clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
      rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
      Warning: apiregistration.k8s.io/v1beta1 APIService is deprecated in v1.19+, unavailable in v1.22+; use apiregistration.k8s.io/v1 APIService
      apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
      serviceaccount/metrics-server created
      deployment.apps/metrics-server created
      service/metrics-server created
      clusterrole.rbac.authorization.k8s.io/system:metrics-server created
      clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
      ```

2. Ensure the install is working

  ```kubectl get --raw /apis/metrics.k8s.io/```

    - Output:
      ```
      {"kind":"APIGroup","apiVersion":"v1","name":"metrics.k8s.io","versions":[{"groupVersion":"metrics.k8s.io/v1beta1","version":"v1beta1"}],"preferredVersion":{"groupVersion":"metrics.k8s.io/v1beta1","version":"v1beta1"}}
      ```

3. Create a pod to moinitor

  ```kubectl apply -f pod.yml```
  *Note: this pod was pre-exsisting*

4. Next get the metric data

  ```kubectl top pods```
    - Output:
      ```
      NAME     CPU(cores)   MEMORY(bytes)
      my-pod   0m           0Mi
      ```

5. Example on how to sort by metric data

  ```kubectl top pod --sort-by cpu```

6. Example on how to use a selector

  ```kubectl top pod --selector app=metrics-test```
  *Note: this selector was defined in the creation of the pod YAML file*

7. How to check resource usage by node

  ```kubectl top node```
    - Output:
      ```
      NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
      instance-1-cka   164m         8%     1124Mi          29%
      instance-2-cka   78m          3%     656Mi           17%
      instance-3-cka   62m          3%     888Mi           23%
      ```


