# Upgrading K8s with kubeadm

### Relevant Docs:
  - [Upgrading kubeadm Clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

  ---

## Lesson Reference

1. First, upgrade the control plane node.
  - Drain the conrol plane node.
  ```
  kubectl drain <control plane node name> --ignore-daemonsets
  ```
  - Upgrade kubeadm.
  ```
  sudo apt-get update && \
  sudo apt-get install -y --allow-change-held-packages kubeadm=1.20.2-00

  kubeadm version
  ```
  - Plan the upgrade.
  ```
  sudo kubeadm upgrade plan v1.20.2
  ```
  - Upgrade the control plane components.
  ```
  sudo kubeadm upgrade apply v1.20.2
  ```
  - Upgrade kubelet and kubectl on the conrol plane node.
  ```
  sudo apt-get update && \
  sudo apt-get install -y --allow-change-held-packages kubelet=1.20.2-00 kubectl=1.20.2-00
  ```
  - Restart kubelet.
  ```
  sudo systemctl daemon-reload

  sudo systemctl restart kubelet
  ```
  - Uncordon the control plane node.
  ```
  kubectl uncordon <control plane node name>
  ```
  - Verify that the control plane is working.
  ```
  kubectl get nodes
  ```

2. Upgrade the worker nodes
    - *Note: In a real-world scenario, you should not perform upgrades on all worker nodes at the same time. Make sure enough nodes are available at any given time to provide uninterrupted service.*

  - **Run the following on the control plane node** to drain worker node 1:
  ```
  kubectl drain <worker 1 node name> --ignore-daemonsets --force
  ```
  - Log in to the first worker node, then Upgrade kubeadm
  ```
  sudo apt-get update && \
  sudo apt-get install -y --allow-change-held-packages kubeadm=1.20.2-00

  kubeadm version
  ```
  - Upgrade the kubelet configuration on the worker node.
  ```
  sudo kubeadm upgrade node
  ```
  - Upgrade kubelet and kubectl on the worker node.
  ```
  sudo apt-get update && \
  sudo apt-get install -y --allow-change-held-packages kubelet=1.20.2-00 kubectl=1.20.2-00
  ```
  - Restart kubelet.
  ```
  sudo systemctl daemon-reload

  sudo systemctl restart kubelet
  ```
  - **From the control plane node**, uncordon worker node 1.
  ```
  kubectl uncordon <worker 1 node name>
  ```
3. Repeat the upgrade process for worker node 2

  - **From the control plane node**, drain worker node 2.
  ```
  kubectl drain <worker 2 node name> --ignore-daemonsets --force
  ```
  - On the second worker node, upgrade kubeadm
  ```
  sudo apt-get update && \
  sudo apt-get install -y --allow-change-held-packages kubeadm=1.20.2-00

  kubeadm version
  ```
  - Perform the same upgrade on worker node 2.
  ```
  sudo kubeadm upgrade node

  sudo apt-get update && \
  sudo apt-get install -y --allow-change-held-packages kubelet=1.20.2-00 kubectl=1.20.2-00

  sudo systemctl daemon-reload

  sudo systemctl restart kubelet
  ```
  - **From the control plane node**, uncordon worker node 2.
  ```
  kubectl uncordon <worker 2 node name>
  ```
  - Verify that the cluster is upgraded and working.
  ```
  kubectl get nodes
  ```