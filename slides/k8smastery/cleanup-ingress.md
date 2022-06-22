class: cleanup

## Cleanup

Let's cleanup before we start the next lecture!

.exercise[

- remove our ingress controller:
  ```bash
  # for Docker Desktop with LoadBalancer
  kubectl delete -f https://k8smastery.com/ic-traefik-lb.yaml
  # for minikube/MicroK8s with hostNetwork
  kubectl delete -f https://k8smastery.com/ic-traefik-hn.yaml
  ```

- remove our ingress resources:
  ```bash
  kubectl delete -f ingress.yaml
  kubectl delete -f https://k8smastery.com/redirect.yaml
  ```

- remove our cheeses:
  ```bash
  kubectl delete svc/cheddar svc/stilton svc/wensleydale 
  kubectl delete deploy/cheddar deploy/stilton deploy/wensleydale
  ```

]
