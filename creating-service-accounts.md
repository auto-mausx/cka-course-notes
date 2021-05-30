# Creating Service Accounts

> RBAC is a powerful way to manage access to Kubernetes resources for regular user accounts. However, you can also use it to manage access from within your Pods. ServiceAccounts allow you to control the ways in which Pods in the cluster can utilize the Kubernetes API. This lesson will focus on the process of creating and binding roles to ServiceAccounts.

## Relevant Docs:

- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## Lesson Reference

1. Create a basic ServiceAccount.

  ```vi my-serviceaccount.yml```

  ```YAML
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-serviceaccount
  ```

  ```kubectl create -f my-serviceaccount.yml```

2. Create a ServiceAccount with an imparative command

  ```kubectl create sa my-serviceaccount2 -n default```

3. View your ServiceAccount

  ```kubectl get sa```

4. Attach a Role to the ServiceAccount with a RoleBinding

  ```vi sa-pod-reader.yml```

  ```YAML
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: sa-pod-reader
    namespace: default
  subjects:
  - kind: ServiceAccount
    name: my-serviceaccount
    namespace: default
  roleRef:
    kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
    ```

    ```kubectl create -f sa-pod-reader.yml```

5. Get additional info for the ServiceAccount

  ```kubectl describe sa my-serviceaccount```

---

## What is a Service Account?

  > In K8s, a **service account** is an account used by container processes within Pods to authenticate with the K8s API.

  > If your Pods need to communicate with the K8s API, you can use service accounts to control their access.

## Creating Service Accounts

  > A service account object can be created with some YAML just like any other K8s object.

  ```YAML
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-serviceaccount
  ```

## Binding Roles to Service Accounts

  > You can manage access control for service accounts, just like any other user, using **RBAC** objects.

  > Bind service accounts with ClusterRoles or ClusterRoleBindings to provide access to K8s API functionality

  ```YAML
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: sa-pod-reader
  subjects:
  - kind: ServiceAccount
    name: my-serviceaccount
    namespace: default
  roleRef:
    kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
  ```

## Hands-On Demo
  *See Lesson Reference above*