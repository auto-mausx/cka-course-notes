# Intro to K8s Management Tools

> Managing Kubernetes in the real world can come with challenges. Luckily, there are companion tools available to make your life easier when running and managing Kubernetes applications. In this lesson, we briefly explore a few of these options.

### Relevant Docs

- [kustomize](https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/)

- [k8s Tools](https://kubernetes.io/docs/reference/tools/)

---

## K8s Management Tools

> There is a variety of management tools available for K8s. These tools interface with K8s to provide additional functionality. When using K8s, it is a good idea to be aware of some of these tools.

## kubectl

> kubectl is the official command line interface for K8s. It is the main method you will use to work with K8s in this course (and the exam).


## kubeadm

> kubeadm is a tool for quickly and easily creating K8s clusters.

## Minikube

> Minikube allows you to automatically set up a lical single-node K8s cluster. It's great for getting K8s up and running quickly for dev purposes.

## Helm

> Helm provides templating and package management for K8s objects. You can use it to manage your own templates (charts). You can also download and use shared templates.

## Kompose

> Kompose helps you translate Docker compose files into K8s objects. If you are using Docker compose for some part of your workflow, you can move your application to K8s easily with Kompose.

## Kustomize

> Kustomize is a configuration management tool for managing K8s object configurations. It allows you to share and re-use templated configurations for K8s applications.