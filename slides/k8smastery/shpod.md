name: shpod

## `shpod`: For a consistent Kubernetes experience ...

- You can use [shpod](https://github.com/bretfisher/shpod) for examples

- `shpod` provides a shell running in a pod on the cluster

- It comes with many tools pre-installed (helm, stern, curl, jq...)

- These tools are used in many exercises in these slides

- `shpod` also gives you shell completion and a fancy prompt

- Create it with `kubectl apply -f https://k8smastery.com/shpod.yaml`

- Attach to shell with `kubectl attach --namespace=shpod -ti shpod`

- After finishing course `kubectl delete -f https://k8smastery.com/shpod.yaml`

