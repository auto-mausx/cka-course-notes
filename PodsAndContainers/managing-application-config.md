# Managing Application Configuration

> There are many ways to configure applications in Kubernetes. In this lesson, we will examine some of the Kubernetes features that can help manage application configuration. We will demonstrate how to store configuration data in the form of ConfigMaps and Secrets, and we will discuss options for passing that data to containers.

## Relevant Docs:

- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)

- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

---

## Application Configuration

> When you are running applications in K8s, you may want to pass dynamic values to your applications at runtime to control how they behave. This is known as **application configuration**.

## Config Maps

> You can store configuration data in K8s using **ConfigMaps**. ConfigMaps store data in the form of a key-value map. ConfigMap data can be passed to your container applications.

  - Example:

  ```YAML
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-configmap
  data:
    key1: value1
    key2: value2
    key3:
      subkey:
        morekeys: data
        evenmore: some more data
    key4: |
      You can also do
      multi-line
      data.
  ```

## Secrets

> **Secrets** are similar to ConfigMaps, but are desinged to store sensitive data, such as passwords or API keys, more securely. They are created and used similarly to ConfigMaps.

  - Example:

  ```YAML
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-secret
  type: Opaque
  data:
    username: user
    password: mypass
  ```

## Env Variables

> You can pass ConfigMap and Secret data to your containers as **environment variables**. These variables will be visible in your container process at runtime

  - Example:

  *Note: part of pod specification file container spec*

  ```YAML
  spec:
    containers:
    - ...
      env:
      - name: ENVVAR
        valueFrom:
          configMapKeyRef:
            name: my-configmap
            key: mykey
  ```

## Config Volumes

> Configuration data from ConfigMaps and Secrets can also be passed to containers in the form of **mounted volumes**. This will cause the configuration data to appear in files available to the container file system.

> Each top-level key in the configuration data will appear as a file containing all keys below that top-level key.

  - Example:
    *Note: check hands on demo section for more info*

  ```YAML
  ...
  volumes:
  - name: secret-vol
    secret:
      secretName: my-secret
  ```

## Hands-On Demo

1. Create ConfigMap file:

  ```YAML
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-configmap
  data:
    key1: Hello, world!
    key2: |
      Test
      multiple lines
      more lines
  ```

2. Create ConfigMap using

  `kubectl create -f my-configmap.yml`

  - Output:
  `configmap/my-configmap created`

3. Use `kubectl describe configmap my-configmap` to see data

  - Output:
  ```
  Name:         my-configmap
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>

  Data
  ====
  key1:
  ----
  Hello, world!
  key2:
  ----
  Test
  multiple lines
  more lines

  Events:  <none>
  ```

4. Create a Secret via YAML

  ```YAML
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-secret
  type: Opaque
  data:
    secretkey1:
    secretkey2:
  ```

  *Note: you need to base64 encode the two secret keys when using yaml file to create secrets*

5. Encode secrets

  `echo -n 'secret' | base64`

  - Output:
    `c2VjcmV0`

6. Open my-secret.yml again and place the base64 value into secretkey1 value

  ```vi my-secret.yml```

  ```YAML
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-secret
  type: Opaque
  data:
    secretkey1: c2VjcmV0
    secretkey2:
  ```

7. Repeat Process for secretkey2

  ```echo -n 'anothersecret' | base64```

  ```vi mysecret.yml```

  ```YAML
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-secret
  type: Opaque
  data:
    secretkey1: c2VjcmV0
    secretkey2: YW5vdGhlcnNlY3JldA==
  ```

8. Create the secret `my-secret`

  ``` kubectl create -f my-secret.yml ```

  - Output:
  ``` secret/my-secret created ```

  ### Next we need to pass the secrets via ENV variables

9. Create pod definition file for env variables

  ``` vi env-pod.yml ```

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: env-pod
  spec:
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'echo "configmap: $CONFIGMAPVAR secret: $SECRETVAR"']
      env:
      - name: CONFIGMAPVAR
        valueFrom:
          configMapKeyRef:
            name: my-configmap
            key: key1
      - name: SECRETVAR
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: secretkey1
  ```

10. Create the pod

  ``` kubectl create -f env-pod.yml ```

  - Output:
    ``` pod/env-pod created ```

11. Check to see if ENV variables are passed through

  ``` kubectl logs env-pod ```

  - Output:
    ``` configmap: Hello, world! secret: secret ```

### Next we will use configuration volumes to do the same

12. Create config volume yaml file: `volume-pod.yml`

  ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: volume-pod
  spec:
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'while true; do not sleep 3600; done']
      volumeMounts:
      - name: configmap-volume
        mountPath: /etc/config/configmap
      - name: secret-volume
        mountPath: /etc/config/secret
    volumes:
    - name: configmap-volume
      configMap:
        name: my-configmap
    - name: secret-volume
      secret:
        secretName: my-secret
  ```

13. Create the pod

  ``` kubectl create -f volume-pod.yml ```

  - Output:
    ``` pod/volume-pod created ```

14. Verify you can see config data in pod

  ``` kubectl exec volume-pod -- ls /etc/config/configmap ```

  - Output:
    ```
    key1
    key2
    ```

*Note: you can now see that those top level keys have been created as a file on the pod*

15. Check contents of the files

  ``` kubectl exec volume-pod -- cat /etc/config/configmap/key1 ```

  - Output:
    ``` Hello, world! ```

  ``` kubectl exec volume-pod -- cat /etc/config/configmap/key1 ```

  - Output:
    ```
    Test
    multiple lines
    more lines
    ```

  ``` kubectl exec volume-pod -- ls /etc/config/secret ```

  - Output:
    ```
    secretkey1
    secretkey2
    ```

  ``` kubectl exec volume-pod -- cat /etc/config/secret/secretkey1 ```

  - Output:
    ``` secret ```

  ``` kubectl exec volume-pod -- cat /etc/config/secret/secretkey2 ```

  - Output:
    ``` anothersecret ```

