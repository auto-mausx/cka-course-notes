# Intro to High Availability in K8s

> High availability is an important topic for any resilient system in the real world. If your Kubernetes cluster is not highly available, it is likely the applications running on it aren’t either. In this lesson, we will discuss some options for building a highly available Kubernetes infrastructure.

### Relevant Docs

- [Options for Highly Available Topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/)

---

## High Availability Control Plane

> K8s facilitates high-availability applications, but you can also design the cluster itself to be highly available.

> To do this, you need multiple control plane nodes.

  - When using multiple conrol planes for high availability, you will likely need to communicate with the K8s API through a load balancer.

  - This includes clients such as `kubelet` instances running on worker nodes

    - Diagram:
    ![alt text](./CKA-diagrams/CKA-HACP.png)

## Stacked etcd

- etcd runs on the same servers/nodes as the rest of the control plane components.

- kubeadm uses this methodology (this course)

    - Diagram:
    ![alt text](./CKA-diagrams/CKA-stacked_etcd.png)


## External etcd

- etcd runs on completely separate servers/nodes

- we can have multiple etcd nodes in a HA (high availability) cluster which would be a completely different set of servers that run our normal K8s control plane components.

- with external etcd you can have any number of K8s control plane instances and any number of etcd nodes.

    - Diagram:
    ![alt text](./CKA-diagrams/CKA-external_etcd.png)

