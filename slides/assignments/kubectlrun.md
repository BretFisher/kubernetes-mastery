# Assignment: first steps

1. Cluster inventory

   1.1. How many nodes does your cluster have?

   1.2. What kernel version and what container engine is each node running?

2. Control plane examination

   2.1. List the pods in the `kube-system` namespace.

   2.2. Explain the role of some of these pods.

   2.3. If there are no pods in `kube-system`, why could that be?

3. Running containers

   3.1. Run a container using image `jpetazzo/clock`.

   3.2. Run two more containers using that same image.

   3.3. Show the last line of output of these three containers.

---

class: answers

## Answers

1.1. We can get a list of nodes with `kubectl get nodes`.

If our cluster has many nodes and we want to count them accurately, we could use:

`kubectl get nodes -o name | wc -l`.

1.2. `kubectl get nodes -o wide` will list extra information for each node.

This will include kernel version and container engine.

---

class: answers

## Answers

2.1. `kubectl get pods --namespace=kube-system` lists pods in the `kubesystem` namespace.

2.2. This depends on how our cluster was set up.

On some clusters, we might see pods named `etcd-XXX`, `kube-apiserver-XXX`: these correspond to control plane components.

It's also common to see `kubedns-XXX` or `coredns-XXX`: these implement the DNS service that lets us resolve service names into their ClusterIP address.

2.3. On some clusters, the control plane is located *outside* the cluster itself.

In that case, the control plane won't show up in `kube-system`.

---

class: answers

## Answers

3.1. `kubectl run ticktock --image=jpetazzo/clock`

This will create a *Deployment* called *ticktock*.

By default, it will have one replica, translating to one container.

3.2. `kubectl scale deployment ticktock --replicas=3`

This will scale the deployment to three replicas (two more containers).

3.3. `kubectl logs --selector=run=ticktock --tail=1`

All the resources created with `kubectl run xxx` will have the label `run=xxx`.

Therefore, we use the selector `run=ticktock` here to match all the pods belonging to this deployment.
