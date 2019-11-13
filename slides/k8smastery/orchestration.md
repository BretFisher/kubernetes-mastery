# What and why of orchestration

- There are many computing orchestrators

- They make decisions about when and where to "do work"

--

- We've done this since the dawn of computing: Mainframe schedulers, Puppet, Terraform, AWS, Mesos, Hadoop, etc.

--

- Since 2014 we've had a resurgence of new orchestration projects because:

--

  1. Popularity of distributed computing

--

  2. Docker containers as a app package and isolated runtime

--

- We needed "many servers to act like one, and run many containers"

--

- An the Container Orchestrator was born

---

## Container orchestrator

- Many open source projects have been created in the last 5 years to:

  - Schedule running of containers on servers

--

  - Dispatch them across many nodes

--

  - Monitor and react to container and server health

--

  - Provide storage, networking, proxy, security, and logging features

--

  - Do all this in a declarative way, rather than imperative

--

  - Provide API's to allow extensibility and management

---

## Major container orchestration projects

- Kubernetes, aka K8s

- Docker Swarm (and Swarm classic)

- Apache Mesos/Marathon

- Cloud Foundry

- Amazon ECS (not OSS, AWS-only)

- HashiCorp Nomad


--

- **Many of these tools run on top of Docker Engine**

--

- **Kubernetes is the *one* orchestrator with many _distributions_**

---

## Kubernetes distributions

- Kubernetes "vanilla upstream" (not a distribution)

--

- Cloud-Managed distros: AKS, GKE, EKS, DOK...

--

- Self-Managed distros: RedHat OpenShift, Docker Enterprise, Rancher, Canonical Charmed, openSUSE Kubic...

--

- Vanilla installers: kubeadm, kops, kubicorn...

--

- Local dev/test: Docker Desktop, minikube, microK8s

--

- CI testing: kind

--

- Special builds: Rancher k3s

--

- And [Many, many more...](https://kubernetes.io/partners/#conformance) (86 as of June 2019)

