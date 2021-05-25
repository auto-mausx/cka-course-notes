# Kubectl Tips

> When taking the CKA exam, you will need to be able to work with kubectl quickly in order to complete the exam tasks within a limited time frame. In this lesson, we will discuss a few additional tips that will help you use kubectl more efficiently. These tips will help speed up your interactions with Kubernetes and allow you more time on the exam to think through the questions.

## Relevant Docs
  - [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
  - [Object Management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)
  - [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Lesson reference

- Run kubectl create to see a list of objects that can be created with imperative commands.

```kubectl create```

- Create a deployment imperatively.

```kubectl create deployment my-deployment --image=nginx```

- Do a dry run to get some sample yaml without creating the object.

```kubectl create deployment my-deployment --image=nginx --dry-run -o yaml```

- Save the yaml to a file.

```kubectl create deployment my-deployment --image=nginx --dry-run -o yaml > deployment.yml```

- Create the object using the file.

```kubectl create -f deployment.yml```

- Scale a deployment and record the command.

```
kubectl scale deployment my-deployment --replicas=5 --record

kubectl describe deployment my-deployment
```

---

## Imperative Commands

- So far in this course we have been using Declarative format. Creating YAML files, then using `kubectl create` to create the pods. Defining objects using data structures such as YAML or JSON

- Imperative defines objects using kubectl commands and flags. Some people find imperative commands faster.

```
kubectl create deployment my-deployment -- image=nginx
```

## Quick Sample YAML

- Use the `--dry-run` flag to run an imperative command without creating an object. Combine it with `-o yaml` to quickly obtain a sample YAML file you can manipulate.

```
$ kubectl create deployment my-deployment --image=nginx --dry-run -o yaml
```
## Record a Command

- Use the `--record` flag to record the command that was used to make a change.

```
$ kubectl scale deployment my-deployment replicas=5 --record
```

## Use the Docs

- You can often find YAML examples in the K8s docs. You are allowed to use this documenation during the exam. Feel free to copy and paste example YAML and/or commands from the docs.

## Hands on Demo

**See lesson reference above for demo**