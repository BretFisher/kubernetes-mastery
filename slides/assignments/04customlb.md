name: assignment4

# Assignment 4: custom load balancing

Our goal here will be to create a service that load balances
connections to two different deployments. You might use this as a 
simplistic way to run two versions of your apps in parallel. 

In the real world, you'll likely use a 3rd party load balancer to 
provide advanced blue/green or canary-style deployments, but 
this assignment will help further your understanding of how service 
selectors are used to find pods to use as service endpoints.

For simplicity, version 1 of our application will be using
the NGINX image, and version 2 of our application will be using
the Apache image. They both listen on port 80 by default.

When we connect to the service, we expect to see some requests
being served by NGINX, and some requests being served by Apache.

---

## Hints

We need to create two deployments: one for v1 (NGINX), another for v2 (Apache).

--

They will be exposed through a single service.

--

The *selector* of that service will need to match the pods created
by *both* deployments.

--

For that, we will need to change the deployment specification to add
an extra label, to be used solely by the service.

--

That label should be different from the pre-existing labels of our
deployments, otherwise our deployments will step on each other's toes.

--

We're not at the point of writing our own YAML from scratch, so you'll 
need to use the `kubectl edit` command to modify existing resources.

---

## Deploying version 1

1.1. Create a deployment running one pod using the official NGINX image.

1.2. Expose that deployment.

1.3. Check that you can successfully connect to the exposed service.

---

## Setting up the service

2.1. Use a custom label/value to be used by the service. How about `myapp: web`.

2.2. Change (edit) the service definition to use that label/value.

2.3. Check that you *cannot* connect to the exposed service anymore.

2.4. Change (edit) the deployment definition to add that label/value to the pods.

2.5. Check that you *can* connect to the exposed service again.

---

## Deploying version 2

3.1. Create a deployment running one pod using the official Apache image.

3.2. Change (edit) the deployment definition to add the label/value picked previously.

3.3. Connect to the exposed service again.

(It should now yield responses from both Apache and NGINX.)

---

class: answers

## Answers

1.1. `kubectl create deployment v1-nginx --image=nginx`

1.2. `kubectl expose deployment v1-nginx --port=80` or `kubectl create service v1-nginx --tcp=80`

1.3.A If you are using `shpod`, or if you are running directly on the cluster:

```bash
### Obtain the ClusterIP that was allocated to the service
kubectl get svc v1-nginx
curl http://`A.B.C.D`
```

1.3.B You can also run a program like `curl` in a container:

```bash
kubectl run --restart=Never --image=alpine -ti --rm testcontainer
### Then, once you get a prompt, install curl
apk add curl
### Then, connect to the service
curl v1-nginx
```

---

class: answers

## Answers

2.1. Edit the YAML manifest of the service with `kubectl edit service v1-nginx`. Look for the `selector:` section, and change `app: v1-nginx` to `myapp: web`. Make sure to change the `selector:` section, not the `labels:` section! After making the change, save and quit.

2.2. The `curl` command (see previous slide) should now time out.

2.3. Edit the YAML manifest of the deployment with `kubectl edit deployment v1-nginx`. Look for the `labels:` section **within the `template:` section**, as we want to change the labels of the pods created by the deployment, not of the deployment itself. Make sure to change the `labels:` section, not the `matchLabels:` one. Add `myapp: web` just below `app: v1-nginx`, with the same indentation level. After making the change, save and quit. We need both labels here, unlike the service selector. The app label keeps the pod "linked" to the deployment/replicaset, and the new one will cause the service to match to this pod.

2.4. The `curl` command should now work again. (It might need a minute, since changing the label will trigger a rolling update and create a new pod.)

---

class: answers

## Answers

3.1. `kubectl create deployment v2-apache --image=httpd`

3.2. Same as previously: `kubectl edit deployment v2-apache`, then add the label `myapp: web` below `app: v2-apache`. Again, make sure to change the labels in the pod template, not of the deployment itself.

3.3. The `curl` command show now yield responses from NGINX and Apache.

(Note: you won't see a perfect round-robin, i.e. NGINX/Apache/NGINX/Apache etc., but on average, Apache and NGINX should serve approximately 50% of the requests each.)

