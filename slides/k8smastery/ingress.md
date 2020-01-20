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

## Ingress Diagram

---

## ingress vs. Ingress

**ingress**

- ingress definition: Going in, entering. The opposite of egress (leaving)

- In networking terms, ingress refers to handling incoming connections

- Could imply incoming to firewall, network, or in this case, a server cluster

--

**Ingress**

- Ingress (capital I) in these slides means the Kubernetes Ingress resource

- Specific to HTTP/S

---

## Principle of operation

- Step 1: deploy an *Ingress controller*

  - Ingress controller = load balancing proxy + control loop

  - the control loop watches over Ingress resources, and configures the LB accordingly

  - these might be two separate processes (NGINX sever + NGINX Ingress controller)

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

# Ingress in action: NGINX

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

  minikube and MicroK8s don't work with it

  ... but Docker Desktop supports it for `localhost`!

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

---

## What *you* will use now

- Docker Desktop:

  - no built-in Ingress installer, we'll provide you YAML

  - Ignores `hostNetwork`, but Service `type: LoadBalancer` works with `localhost`!

--

- minikube:

  - has a built-in NGINX installer `minikube addons enable ingress`

  - But, let's use YAML we provide for learning purposes

  - `hostNetwork: true` enabled on pod works for minikube IP
 
--

- MicroK8s:

  - has a built-in NGINX installer `microk8s.enable ingress`
  
  - let's use YAML we provide anyway for learning purposes
  
  - `hostNetwork: true` enabled on pod works for minikube IP

---

## First steps with NGINX

- Remember the three parts of Ingress:

  - Ingress controller pod(s) to monitor the API and run the LB/proxy

  - Ingress Resources that tell the LB where to route traffic

  - Services for your apps so the Ingress LB/proxy can route to your pods

- First, lets apply YAML to create the Ingress controller

---

## Deploying the NGINX Ingress controller

- We need the YAML templates from the [kubernetes/ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/) project

The two main parts are:

--
  
  - NGINX Deployment (or DaemonSet) and all its resources

    - Namespace
    - ConfigMaps (storing NGINX configs)
    - ServiceAccount (authenticate to Kubernetes API)
    - Role/ClusterRole/RoleBindings (authorization to API parts)
    - LimitRange (limit cpu/memory of NGINX)

--
 
  - Service to expose NGINX on 80/443

    - different for each Kubernetes distribution

---

## Running NGINX on our cluster

- Now let's deploy the NGINX controller. Pick your distro:

.exercise[

- Apply the YAML
  ```bash
  # for Docker Desktop, create Service with LoadBalancer
  kubectl apply -f https://k8smastery.com/ic-nginx-lb.yaml
  
  # for minikube/MicroK8s, create Service with hostNetwork
  kubectl apply -f https://k8smastery.com/ic-nginx-hn.yaml
  ```
- Check the pod Status
  ```bash
  kubectl describe -n ingress-nginx deploy/nginx-ingress-controller
  ```
]

---

## Checking that NGINX runs correctly

- If NGINX started correctly, we now have a web server listening on each node

.exercise[

- Direct your browser to your Kubernetes IP on port 80

]

We should get a `404 page not found` error.

This is normal: we haven't provided any Ingress rule yet.

---

## Setting up DNS

- To make our lives easier, we will use [nip.io](http://nip.io)

- Check out `http://cheddar.A.B.C.D.nip.io`

  (replacing A.B.C.D with the IP address of your Kubernetes IP)

- We should get the same `404 page not found` error

  (meaning that our DNS is "set up properly", so to speak!)

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

- Edit the file `ingress.yaml` which has three Ingress resources

- Replace the A.B.C.D with your Kubernetes IP (`127.0.0.1` for `localhost`)

- Apply the file `kubectl apply -f ingress.yaml`

- Open http://cheddar.A.B.C.D.nip.io

]

(An image of a piece of cheese should show up.)

---

## Bring up the other Ingress resources

.exercise[

- Open http://stilton.A.B.C.D.nip.io

- Open http://wensleydale.A.B.C.D.nip.io

]

- Different cheeses should show up for each URL

---

## Adding features to a Ingress resource

- Reverse proxies have lots of features

- Let's add a 301 redirect to a new Ingress resource using annotations

- It will apply when any other path is used in URL that we didn't already add

.exercise[

- Create a redirect

```bash
kubectl apply -f https://k8smastery.com/redirect.yaml
```

- Open http://whatever.A.B.C.D.nip.io or localhost or A.B.C.D
]

- It should immediately redirect to google.com

---

## Annotations can get weird and complex

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-google
  annotations:
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.google.com
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: doesntmatter
          servicePort: 80
```

- Ingress resource insists we have a rule and backend, even though it's not needed here

---

# Swapping NGINX for Traefik 1.x

- Traefik is another Ingress controller option

- Most importantly: Traefik releases are named after cheeses 🧀🎉

--

- The [Traefik documentation](https://docs.traefik.io/v1.7/user-guide/kubernetes/#deploy-trfik-using-a-deployment-or-daemonset)
  tells us to pick between Deployment and DaemonSet

- We are going to use a DaemonSet so that each node can accept connections

--

- We provide a YAML file which is essentially the sum of:

  - [Traefik's DaemonSet resources](https://github.com/containous/traefik/blob/v1.7/examples/k8s/traefik-ds.yaml) (patched with `hostNetwork` and tolerations)

  - [Traefik's RBAC rules](https://github.com/containous/traefik/blob/v1.7/examples/k8s/traefik-rbac.yaml) allowing it to watch necessary API objects

- We will make a minor change to the 
  [YAML provided by Traefik](https://github.com/containous/traefik/blob/v1.7/examples/k8s/traefik-ds.yaml)
  to enable `hostNetwork` for MicroK8s/minikube

- For Docker Desktop we'll add a `type: LoadBalancer` to the Service

---

## Removing NGINX from our cluster

- Before starting Traefik, let's remove the NGINX controller

- This won't remove Services or Ingress resources

- But it will make them unavailble from outside the cluster

.exercise[

- Delete our NGINX controler and related resources:
  ```bash
  # for Docker Desktop with LoadBalancer
  kubectl delete -f https://k8smastery.com/ic-nginx-lb.yaml

  # for minikube/MicroK8s with hostNetwork
  kubectl delete -f https://k8smastery.com/ic-nginx-hn.yaml
  ```

]

--

- Wait 30 seconds for everything to cleanup

---

## Running Traefik 1.x on our cluster

- Now let's deploy the Traefik 1.x controller

.exercise[

- Apply the YAML:
  ```bash
  # for Docker Desktop with LoadBalancer
  kubectl apply -f https://k8smastery.com/ic-traefik-lb.yaml
  
  # for minikube/MicroK8s with hostNetwork
  kubectl apply -f https://k8smastery.com/ic-traefik-hn.yaml
  ```
- Check the pod Status
  ```bash
  kubectl describe -n kube-system ds/traefik-ingress-controller
  ```
]

---

## Checking that Traefik runs correctly

- If Traefik started correctly, we can refresh a cheese and it still works

.exercise[

- Refresh http://cheddar.A.B.C.D.nip.io

]

--

- Notice the redirect Ingress resource doesn't work (because it's NGINX specific)

.exercise[

- `curl localhost` from host

- You should get back `404 page not found`

Note if you try the browser, it may cache the redirect response and still redirect
]

---

## Traefik web UI

- Traefik provides a web dashboard

- With the current install method, it's listening on port 8080

.exercise[

- Go to `http://<Kubernetes IP>:8080`

<!-- ```open http://node1:8080``` -->

]

---

## What about Traefik 2.x?

- We've been using Traefik 1.x as the Ingress controller

- Traefik released 2.0 in late 2019

--

- But it uses a custom CRD (Custom Resource Definition), not Ingress resources

- We'll explore why in a bit

---

## Using multiple ingress controllers

- You can have multiple ingress controllers active simultaneously

  (e.g. Traefik, Gloo, and NGINX)

--

- You can even have multiple instances of the same controller

  (e.g. one for internal, another for external traffic)

--

- The `kubernetes.io/ingress.class` annotation can be used to tell which one to use

--

- It's OK if multiple ingress controllers configure the same resource

  (it just means that the service will be accessible through multiple paths)

--

- TCP/IP IP:PORT rules still apply: Only one can bind to 80 on host IP

---

## Ingress resources: the good

- The traffic flows directly from the ingress load balancer to the backends

  - it doesn't need to go through the `ClusterIP`

  - in fact, we don't even need a `ClusterIP` (we can use a headless service)

--

- The load balancer can be outside of Kubernetes

  (as long as it has access to the cluster subnet)

- This allows the use of external (hardware, physical machines...) load balancers

--

- Annotations can encode special features

  (rate-limiting, A/B testing, session stickiness, etc.)

---

## Ingress resources: the bad (*cough* Annotations *cough*)

- Aforementioned "special features" are not standardized yet

- Some controllers will support them; some won't

--

- Even relatively common features (stripping a path prefix) can differ:

  - [traefik.ingress.kubernetes.io/rule-type: PathPrefixStrip](https://docs.traefik.io/user-guide/kubernetes/#path-based-routing)

  - [ingress.kubernetes.io/rewrite-target: /](https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples/rewrite)

--

- This should eventually stabilize

  (remember that ingresses are currently `apiVersion: networking.k8s.io/v1beta1`)

--

- Annotations are not validated in CLI

--

- Some proxies provide a CRD (Custom Resource Definition) option

---

## When not to use built-in Ingress resources

- Your deployment doesn't use alpha or beta features

--

- You need features beyond Ingress including; TCP support, traffic spliting, mTLS, egress, service mesh

--

- You have external load balancers (like AWS ELBs) which route to NodePorts and handle TLS

--

- You don't need externally available HTTP services on the default ports

--

- Your proxy of choice uses a CRD rather then a Ingress Resource

---

## Using CRD's as alternatives to Ingress resources

- Due to the limits of the built-in Ingress, many projects are moving to CRD's

--

- Often called Ingress Gateway or API Gateway, they do ingress plus more:
  - TCP Support (anything beyond HTTP/HTTPS)
  - Traffic splitting, rate limiting, circuit breaking, etc
  - Complex traffic routing, request and reponse transformation

--

- Options include:
  - Envoy Proxy based (Gloo, Ambassador)
  - Reverse Proxy / LB based (NGINX, Traefik, Kong)

--

- Eventually these more advanced features could be added to "Ingress 2.0"

- We'll cover more after we learn about CRD's and Operators
