# A quick example, the power of Kubernetes

- Let's do something easy and fun to start off

--

- Deploy some apps on Kubernetes with a single command!

--

- We'll install our own Kubernetes soon
- For now let's just use a browser-based environment for a quick demo

--

.exercise[

- Open in a new tab to [Katacoda](https://www.katacoda.com/courses/kubernetes/playground)

- Click `Start Scenario`. You have a Kubernetes cluster!

- On "Terminal Host 1 (master)" Let's check our Kubernetes nodes:
  ```bash
  kubectl get nodes 
  ```

- Run a demo app:
  ```bash
  kubectl apply -f https://k8smastery.com/gcp-microservices.yaml
  ```
]

.footnote[.small[
If Katacoda isn't working, you can try [Play-With-K8s](https://labs.play-with-k8s.com/) (which takes a few more steps to setup)
]]

---

## GCP microservices demo

You just launched a [10-tier microservice demo app](https://github.com/GoogleCloudPlatform/microservices-demo) on Kubernetes!

--

- Give it a minute to finish deploying

- Let's get the frontend UI IP address and check it out

.exercise[


```bash
  kubectl get service/frontend-external
```
    
- Browse to that IP on port 8888

]

--

- Cool shopping site, but what did we just deploy?

---

class: pic

![](k8smastery/gcp-microservices-diagram.png)
