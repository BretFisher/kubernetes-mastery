name: assignment1

# Assignment 1: first steps

Answer these questions with the `kubectl` command you'd use to get the answer:

Cluster inventory

   1.1. How many nodes does your cluster have?

   1.2. What kernel version and what container engine is each node running?

(answers on next slide)

---

class: answers

## Answers

1.1. We can get a list of nodes with `kubectl get nodes`.

1.2. `kubectl get nodes -o wide` will list extra information for each node.

This will include kernel version and container engine.

---

## Assignment 1: first steps

Control plane examination

   2.1. List *only* the pods in the `kube-system` namespace.

   2.2. Explain the role of some of these pods.

   2.3. If there are few or no pods in `kube-system`, why could that be?

(answers on next slide)

---

class: answers

## Answers

2.1. `kubectl get pods --namespace=kube-system`

2.2. This depends on how our cluster was set up.

On some clusters, we might see pods named `etcd-XXX`, `kube-apiserver-XXX`: these correspond to control plane components.

It's also common to see `kubedns-XXX` or `coredns-XXX`: these implement the DNS service that lets us resolve service names into their ClusterIP address.

2.3. On some clusters, the control plane is located *outside* the cluster itself.

In that case, the control plane won't show up in `kube-system`, but you can find on host with `ps aux | grep kube`.

---

## Assignment 1: first steps

Running containers

   3.1. Create a deployment using `kubectl create` that runs the image `bretfisher/clock` and name it `ticktock`.

   3.2. Start 2 more containers of that image in the `ticktock` deployment.

   3.3. Use a selector to output only the last line of logs of each container.

(answers on next slide)

---

class: answers

## Answers

3.1. `kubectl create deployment ticktock --image=bretfisher/clock`

By default, it will have one replica, translating to one container.

3.2. `kubectl scale deployment ticktock --replicas=3`

This will scale the deployment to three replicas (two more containers).

3.3. `kubectl logs --selector=app=ticktock --tail=1`

All the resources created with `kubectl create deployment xxx` will have the label `app=xxx`.
If you needed to use a pod selector, you can see them in the resource that created them.
In this case that's the ReplicaSet, so `kubectl describe replicaset ticktock-xxxxx` would help.

Therefore, we use the selector `app=ticktock` here to match all the pods belonging to this deployment.
