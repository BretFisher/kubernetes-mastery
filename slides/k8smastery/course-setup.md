## Doing or re-doing the workshop on your own?

- Use something like
  [Play-With-Docker](http://play-with-docker.com/) or
  [Play-With-Kubernetes](https://training.play-with-kubernetes.com/)

  Zero setup effort; but environment are short-lived and
  might have limited resources

- Create your own cluster (local or cloud VMs)

  Small setup effort; small cost; flexible environments

- Create a bunch of clusters for you and your friends
    ([instructions](https://@@GITREPO@@/tree/master/prepare-vms))

  Bigger setup effort; ideal for group training

---

name: shpod

## `shpod`: For a consistent Kubernetes experience ...

- If you are using your own Kubernetes cluster, you can use [shpod](https://github.com/jpetazzo/shpod)

- `shpod` provides a shell running in a pod on your own cluster

- It comes with many tools pre-installed (helm, stern, curl, jq...)

- These tools are used in many exercises in these slides

- `shpod` also gives you shell completion and a fancy prompt

- Create it with `kubectl apply -f https://bret.run/shpod.yml`

- Attach to shell with `kubectl attach --namespace=shpod -ti shpod`

- Remove with `kubectl delete -f https://bret.run/shpod.yml`



---


