# K8s Networking Architectural Overview

> In this section, we will discuss networking in the context of Kubernetes. We will start by looking at the Kubernetes networking model. You will need to have an understanding of this model to work with networking in Kubernetes.

## Relevant Docs

- [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

---

## Kubernetes Network Model

> The **Kubernetes network model** is a set of standards that define how networking between Pods behaves.

> There are a variety of different implementations of the model - including the **Calico network plugin**, which we have been using throughout this course.

## Network Model Architecture

> The K8s network model dfines how Pods communicate with each other, reguardless of which Node they are running on.

> Each Pod has its own **unique IP address** within the cluster.

> Any Pod can reach any other Pod using that Pod's IP address. This creates a **virtual network** that allows Pods to easily communicate with each other, reguardless of which node they are on.