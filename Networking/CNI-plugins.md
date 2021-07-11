# CNI Plugins Overview

> Kubernetes offers a variety of CNI plugins that implement its networking model in various ways. In this lesson, we discuss some of these options at a high level. We will explore which solution might be preferable in a variety of different situations.

## Relevant Docs

- [Network Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

---

## CNI Plugins

> **CNI plugins** are a type of Kubernetes network plugin. These plugins provide network connectivity between Pods according to the standard set by the Kubernetes network model.

> There are many CNI network plugins available.

## Selecting a Network Plugin

> Which network plugin is best for you will depend on your specific situation.

> Check the K8s docs for a list of available plugins. You may need to research some of these plugins for yourself, depending on your produciton use case.

> The plugin used in this course is Calico.

## Installing Network Plugins

> Each plugin has its own unique installation proess. Early in this course, we installed the Calico network plugin.

  - *Note: Kubernetes nodes will remain `NotReady` until a network plugin is installed. You will be unable to run Pods while this is the case.*