# Exposing HTTP services with Ingress resources

- *Services* give us a way to access a pod or a set of pods

- Services can be exposed to the outside world:

  - with type `NodePort` (on a port >30000)

  - with type `LoadBalancer` (allocating an external load balancer)

- What about HTTP services?

  - how can we expose `webui`, `rng`, `hasher`?

  - the Kubernetes dashboard?

  - a new version of `webui`?

---

## Exposing HTTP services

- If we use `NodePort` services, clients have to specify port numbers

  (i.e. http://xxxxx:31234 instead of just http://xxxxx)

- `LoadBalancer` services are nice, but:

  - they are not available in all environments

  - they often carry an additional cost (e.g. they provision an ELB)
  
  - They often work at OSI Layer 4 (IP+Port) and not Layer 7 (HTTP/S)

  - they require one extra step for DNS integration
    <br/>
    (waiting for the `LoadBalancer` to be provisioned; then adding it to DNS)

- We could build our own reverse proxy

---

## Building a custom reverse proxy

- There are many options available:

  Apache, HAProxy, Envoy Proxy, Gloo, NGINX, Traefik,...

- Most of these options require us to update/edit configuration files after each change

- Some of them can pick up virtual hosts and backends from a configuration store

- Wouldn't it be nice if this configuration could be managed with the Kubernetes API?

--

- Enter.red[¹] *Ingress* resources!

.footnote[.red[¹] Pun maybe intended.]

---

## Ingress resources

- Kubernetes API resource (`kubectl get ingress`/`ingresses`/`ing`)

- Designed to expose HTTP services

- Basic features:

  - load balancing
  - SSL termination
  - name-based virtual hosting

- Can also route to different services depending on:

  - URI path (e.g. `/api`→`api-service`, `/static`→`assets-service`)
  - Client headers, including cookies (for A/B testing, canary deployment...)
  - and more!

---

## ingress vs. Ingress

- ingress definition: Going in, entering. The opposite of egress (leaving)

- In networking terms, ingress refers to handling incoming connections

- Could imply incoming to firewall, network, or in this case, a server cluster

--

- Ingress (capital I) in these slides means the Kubernetes Ingress feature

- Specific to HTTP/S

---

## Principle of operation

- Step 1: deploy an *Ingress controller*

  - Ingress controller = load balancing proxy + control loop

  - the control loop watches over Ingress resources, and configures the LB accordingly

  - these might be two separate pods (NGINX sever + NGINX Ingress controller)

  - or a single app that knows how to speak to Kubernetes API (Traefik)

--

- Step 2: set up DNS

  - associate external DNS entries with the load balancer address

--

- Step 3: create *Ingress resources* for our Service resources

  - these resources contain rules for handling HTTP/S connections

  - the Ingress controller picks up these resources and configures the LB

  - connections to the Ingress LB will be processed by the rules


---

## Ingress in action: NGINX

- We will deploy the NGINX Ingress controller first

  - this is a popular, yet arbitrary choice, the 
  [docs](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
  list over a dozen options

--

- For DNS, we will use [nip.io](http://nip.io/)

  - `*.127.0.0.1.nip.io` resolves to `127.0.0.1`

  - we do this so we can use various FQDN's without editing our `hosts` file

--

- We will create Ingress resources for various HTTP-based Services

---

## Deploying pods listening on port 80

- We want our Ingress load balancer to be available on port 80

--

- We could do that with a `LoadBalancer` service

--

  ... but it requires support from the underlying infrastructure

  For local machine distros, Docker Desktop supports it for `localhost`!

--

- We could use pods specifying `hostPort: 80` 

  ... but with most CNI plugins, this [doesn't work or requires additional setup](https://github.com/kubernetes/kubernetes/issues/23920)

--

- We could use a `NodePort` service

  ... but that requires [changing the `--service-node-port-range` flag in the API server](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

--

- Last resort: the `hostNetwork` mode

---

## Without `hostNetwork`

- Normally, each pod gets its own *network namespace*

  (sometimes called sandbox or network sandbox)

--

- An IP address is assigned to the pod

--

- This IP address is routed/connected to the cluster network

--

- All containers of that pod are sharing that network namespace

  (and therefore using the same IP address)

---

## With `hostNetwork: true`

- No network namespace gets created

--

- The pod is using the network namespace of the host

- It "sees" (and can use) the interfaces (and IP addresses) of the host (VM on macOS/Win)

--

- The pod can receive outside traffic directly, on any port

--

- Downside: with most network plugins, network policies won't work for that pod

  - most network policies work at the IP address level

  - filtering that pod = filtering traffic from the node

--

- Docker Desktop:

  - macOS/Windows `localhost` won't redirect to `hostNetwork`

  - but Service `type: LoadBalancer` works!

---

## First steps with NGINX 

- Remember the three parts of Ingress:

  - Ingress controller pod to monitor the API for new resources

  - Ingress load balancer pod(s) that receive the HTTP/S traffic

  - Ingress Resources that tell the LB where to route traffic

- First, lets apply YAML to create the controller and LB pods 

---

## Deploying the Ingress controller and NGINX proxy

- We need the YAML template from the [kubernetes/ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/) project

- 


---

## Swapping NGINX for Traefik 1.x

- Traefik is another Ingress controller option

- Most importantly: Traefik releases are named after cheeses 🧀🎉

- The [Traefik documentation](https://docs.traefik.io/v1.7/user-guide/kubernetes/#deploy-trfik-using-a-deployment-or-daemonset)
  tells us to pick between Deployment and Daemon Set

- We are going to use a Daemon Set so that each node can accept connections

--

- We will do two minor changes to the 
  [YAML provided by Traefik](https://github.com/containous/traefik/blob/v1.7/examples/k8s/traefik-ds.yaml):

  1. enable `hostNetwork`

--

  2. add a *toleration* so that Traefik also runs on your control plane nodes

  - this isn't an issue on Docker Desktop/MicroK8s/minikube

---

## Taints and tolerations, a preview

- A *taint* is an attribute added to a node

- It prevents pods from running on the node

--

- ... Unless they have a matching *toleration*

--

- When deploying with `kubeadm` and other production-quality distros:

  - a taint is placed on nodes dedicated to the control plane

  - the pods running the control plane have a matching toleration

--

- Each deployment tool will be slightly different in infrastructure design

- Docker Desktop, MicroK8s, and minikube don't add taints

--

- We'll cover this more later

---

## Running Traefik 1.x on our cluster

- We provide a YAML file which is essentially the sum of:

  - [Traefik's Daemon Set resources](https://github.com/containous/traefik/blob/v1.7/examples/k8s/traefik-ds.yaml) (patched with `hostNetwork` and tolerations)

  - [Traefik's RBAC rules](https://github.com/containous/traefik/blob/v1.7/examples/k8s/traefik-rbac.yaml) allowing it to watch necessary API objects
  
  - If using 

.exercise[

- Apply the YAML:
  ```bash
  kubectl apply -f https://k8smastery.com/traefik.yaml
  ```

]

---

## Checking that Traefik runs correctly

- If Traefik started correctly, we now have a web server listening on each node

.exercise[

- Check that Traefik is serving 80/tcp:
  ```bash
  curl localhost
  ```

]

We should get a `404 page not found` error.

This is normal: we haven't provided any ingress rule yet.

---

## Setting up DNS

- To make our lives easier, we will use [nip.io](http://nip.io)

- Check out `http://cheddar.A.B.C.D.nip.io`

  (replacing A.B.C.D with the IP address of `node1`)

- We should get the same `404 page not found` error

  (meaning that our DNS is "set up properly", so to speak!)

---

## Traefik web UI

- Traefik provides a web dashboard

- With the current install method, it's listening on port 8080

.exercise[

- Go to `http://node1:8080` (replacing `node1` with its IP address)

<!-- ```open http://node1:8080``` -->

]

---

## Setting up host-based routing ingress rules

- We are going to use `errm/cheese` images

  (there are [3 tags available](https://hub.docker.com/r/errm/cheese/tags/): wensleydale, cheddar, stilton)

- These images contain a simple static HTTP server sending a picture of cheese

- We will run 3 deployments (one for each cheese)

- We will create 3 services (one for each deployment)

- Then we will create 3 ingress rules (one for each service)

- We will route `<name-of-cheese>.A.B.C.D.nip.io` to the corresponding deployment

---

## Running cheesy web servers

.exercise[

- Run all three deployments:
  ```bash
  kubectl create deployment cheddar --image=errm/cheese:cheddar
  kubectl create deployment stilton --image=errm/cheese:stilton
  kubectl create deployment wensleydale --image=errm/cheese:wensleydale
  ```

- Create a service for each of them:
  ```bash
  kubectl expose deployment cheddar --port=80
  kubectl expose deployment stilton --port=80
  kubectl expose deployment wensleydale --port=80
  ```

]

---

## What does an ingress resource look like?

Here is a minimal host-based ingress resource:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: cheddar
spec:
  rules:
  - host: cheddar.`A.B.C.D`.nip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: cheddar
          servicePort: 80

```

(In [1.14 ingress moved](https://github.com/kubernetes/ingress-gce/issues/770) 
from the extensions API to networking)

---

## Creating our first ingress resources

.exercise[

- Download our YAML `curl -O https://k8smastery.com/ingress.yaml`

- Edit the file `ingress.yaml`

- Replace the A.B.C.D with your cluster IP (`127.0.0.1` for `localhost`)

- Apply the file

- Open http://cheddar.A.B.C.D.nip.io

]

(An image of a piece of cheese should show up.)

---

## Creating the other ingress resources

.exercise[

- Edit the file `ingress.yaml`

- Replace `cheddar` with `stilton` (in `name`, `host`, `serviceName`)

- Apply the file

- Check that `stilton.A.B.C.D.nip.io` works correctly

- Repeat for `wensleydale`

]

---

## Adding features to a ingress resource

- Reverse proxies have lots of features. Let's add one to a ingress resoure

- Let's add a 301 redirect to a new ingress resource

.exercise[

- Edit the file `ingress.yaml`

- Add an annotation to the metadata to pass along to the Ingress Controller

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-google
  annotations:
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.google.com
```

]

---

## Using multiple ingress controllers

- You can have multiple ingress controllers active simultaneously

  (e.g. Traefik, Gloo, and NGINX)

- You can even have multiple instances of the same controller

  (e.g. one for internal, another for external traffic)

- The `kubernetes.io/ingress.class` annotation can be used to tell which one to use

- It's OK if multiple ingress controllers configure the same resource

  (it just means that the service will be accessible through multiple paths)

---

## Ingress: the good

- The traffic flows directly from the ingress load balancer to the backends

  - it doesn't need to go through the `ClusterIP`

  - in fact, we don't even need a `ClusterIP` (we can use a headless service)

- The load balancer can be outside of Kubernetes

  (as long as it has access to the cluster subnet)

- This allows the use of external (hardware, physical machines...) load balancers

- Annotations can encode special features

  (rate-limiting, A/B testing, session stickiness, etc.)

---

## Ingress: the bad (*cough* Annotations *cough*)

- Aforementioned "special features" are not standardized yet

- Some controllers will support them; some won't

- Even relatively common features (stripping a path prefix) can differ:

  - [traefik.ingress.kubernetes.io/rule-type: PathPrefixStrip](https://docs.traefik.io/user-guide/kubernetes/#path-based-routing)

  - [ingress.kubernetes.io/rewrite-target: /](https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples/rewrite)

- This should eventually stabilize

  (remember that ingresses are currently `apiVersion: networking.k8s.io/v1beta1`)

- Annotations are not validated in CLI

- Some proxies provide a CRD (Custom Resource Definition) option

---

## When not to use the Ingress Controller

- Your deployment doesn't use alpha or beta features

- You need features beyond simple Ingress including; TCP support, advanced traffic management like routing/spliting, security like mTLS, egress, and integration to service mesh

- You have external load balancers (like AWS ELBs) which route to NodePorts and handle TLS

- You don't need externally available HTTP services on the default ports

- Your proxy of choice uses a CRD rather then a Ingress Resource
