# Assignment: custom load balancing

Our goal here will be to create a service that load balances
connections to two different versions of an application.

For simplicity, version 1 of our application will be using
the NGINX image, and version 2 of our application will be using
the Apache image. They both listen on port 80 by default.

When we connect to the service, we expect to see some requests
being served by NGINX, and some requests being served by Apache.

Bonus question: we would like the service to be named `janus`
(from the ancient Roman god [Janus](https://en.wikipedia.org/wiki/Janus),
who is often depicted as having two faces).

If you think you know how to do all of that, feel free to go ahead!

Otherwise, the next slide will give us hints, and the following slides
will give us step by step instructions.

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

For the bonus question, we will need to change the name of the service.

--

One way to do that is to dump the YAML of the service, edit it,
and load it anew.

---

## Deploying version 1

1.1. Create a deployment running one pod using the official NGINX image.

1.2. Expose that deployment.

1.3. Check that you can successfully connect to the exposed service.

---

## Setting up the service

2.1. Pick a custom label/value to be used by the service.

2.2. Change the service definition to use that label/value.

2.3. Check that you *cannot* connect to the exposed service anymore.

2.4. Change the deployment definition to add that label/value to the pods.

2.5. Check that you *can* connect to the exposed service again.

---

## Deploying version 2

3.1. Create a deployment running one pod using the official Apache image.

3.2. Change the deployment definition to add the label/value picked previously.

3.3. Connect to the exposed service again.

(It should now yield responses from both Apache and NGINX.)

---

## Changing the name of the service

4.1. Dump the YAML of the service.

4.2. Edit the YAML to change the name of the service.

4.3. Delete the old service.

4.4. Load the new YAML.

4.5. Check that we can connect to the new `janus` service.

---

class: answers

## Answers

1.1. `kubectl create deployment v1-nginx --image=nginx`

1.2. `kubectl expose deployment v1-nginx --port=80`

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

2.1. Our label/value will be `target=janus`.

2.2. Edit the YAML manifest of the service with `kubectl edit service v1-nginx`. Look for the `selector:` section, and change `app: v1-nginx` to `target: janus`. Make sure to change the `selector:` section, not the `labels:` section! After making the change, save and quit.

2.3. The `curl` command (see previous slide) should now time out.

2.4. Change the YAML manifest of the deployment with `kubectl edit deployment v1-nginx`. Look for the `labels:` section **within the `template:` section**, as we want to change the labels of the pods created by the deployment, not of the deployment itself. Make sure to change the `labels:` section, not the `matchLabels:` one. Add `target: janus` just below `app: v1-nginx`, with the same indentation level. After making the change, save and quit.

2.5. The `curl` command should now work again. (It might need a minute, since changing the label will trigger a rolling update and create a new pod.)

---

class: answers

## Answers

3.1. `kubectl create deployment v2-apache --image=httpd`

3.2. Same as previously: `kubectl edit deployment v2-apache`, then add the label `target: janus` below `app: v2-apache`. Again, make sure to change the labels in the pod template, not of the deployment itself.

3.3. The `curl` command show now yield responses from NGINX and Apache.

(Note: you won't see a perfect round-robin, i.e. NGINX/Apache/NGINX/Apache etc., but on average, Apache and NGINX should serve approximately 50% of the requests each.)

---

class: answers

## Answers

4.1. `kubectl get service v1-nginx -o yaml > janus.yaml`

4.2. Edit `janus.yaml` and change `name: v1-nginx` into `name: janus`. 

4.3. `kubectl delete service v1-nginx`

4.4. `kubectl apply -f janus.yaml`

4.5. Check the ClusterIP of the new service.
<br/>If you were testing from a pod, you can just do `curl janus`.

---

## Little details ...

*Why do we need to dump the YAML and load it again?*
<br/>
*Why can't we change the name directly with `kubectl edit service`?*

That's because Kubernetes cannot rename resources. So if it notices that the name was changed during a `kubectl edit` operation, it will bail out. That's why we need to dump the manifest and create a new resource instead.

*Why do we need to delete the old service before creating the new one?*

That's because we cannot have two services with the same ClusterIP. If we try to create another service with that YAML that we dumped, Kubernetes would reject it because it would have the same ClusterIP as the old one.

Another option (if we want to keep both services) is to remove the `clusterIP` line from the YAML before loading it. Kubernetes will then automatically a (random) ClusterIP to the new service.
