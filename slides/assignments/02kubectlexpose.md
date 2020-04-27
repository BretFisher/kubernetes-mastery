name: assignment2

# Assignment 2: first deployment

1. Create a deployment called `littletomcat` with a container using image `tomcat`.

2. Find the IP address of that Tomcat server.

3. Ping it!

  (Use the `shpod` environment if necessary.)

4. What happens if we delete the pod that holds Tomcat?

---

class: answers

## Answers

1. Create a deployment named `littletomcat`:
   ```bash
   kubectl create deployment littletomcat --image=tomcat
   ```
   All resources belonging to that deployment will have the label `app=littletomcat`.

2. List all pods with label `app=littletomcat`, with extra details including IP address:
   ```bash
   kubectl get pods --selector=app=littletomcat -o wide
   ```

3. Start a shell *inside* the cluster:
   ```bash
   curl https://shpod.sh | sh
   ```
   Then the IP address of the pod should ping correctly. Keep the `ping` running.

---

class: answers

## Answers

For question 4, here is how we can delete the pod:

```bash
kubectl delete pods --selector=app=littletomcat
```

The following things will happen:

- the pod will be gracefully terminated,

- the ping command that we left running will fail,

- the replica set will notice that it doens't have the right count of pods and create a replacement pod,

- that new pod will have a different IP address (so the `ping` command won't recover).

---

## Assignment 2: first service

1. How can we give our Tomcat server a stable address?

  (An address that doesn't change when something bad happens to the container.)

2. Create such an address.

3. Check that you can connect to Tomcat with that address.

   (Use the `shpod` environment if necessary.)

4. If we delete again the pod that holds Tomcat, does the address still work?

---

class: answers

## Answers

1. We need to create a *service* for our deployment.

   The service will have a *ClusterIP* which will be usable from within the cluster.

2. One way to create the service is with the `kubectl expose` command:
   ```bash
   kubectl expose deployment littletomcat --port=8080
   ```
   (The Tomcat image is listening on port 8080.)

3. In the `shpod` environment that we started earlier:
   ```bash
   # Install curl
   apk add curl
   # Make a request to the littletomcat service
   curl http://littletomcat:8080
   ```

4. If we delete the pod, another will be created to replace it.
   The *ClusterIP* will still work.

   (Except during a short period while the replacement container is being started.)
