name: assignment2

# Assignment 2: more about deployments

1. Create a deployment called `littletomcat` using the `tomcat` image.

2. What command will help you get the IP address of that Tomcat server?

3. What steps would you take to ping it from another container?

  (Use the `shpod` environment if necessary.)

4. What command would delete the running pod inside that deployment?

5. What happens if we delete the pod that holds Tomcat, while the ping is running?

(answers on next two slides)

---

class: answers

## Answers

1. `kubectl create deployment littletomcat --image=tomcat`

2. List all pods with label `app=littletomcat`, with extra details including IP address:
   `kubectl get pods --selector=app=littletomcat -o wide`. You could also describe 
   the pod: `kubectl describe pod littletomcat-XXX-XXX`

3. Start a shell *inside* the cluster: `curl https://shpod.sh | sh`
   Then the IP address of the pod should ping correctly. You could also start a deployment
   or pod temporarily (like nginx), then exec in, install ping, and ping the IP.

---

class: answers

## Answers

4. We can delete the pod with: `kubectl delete pods --selector=app=littletomcat`
or copy/paste the exact pod name and delete it.

5. If we delete the pod, the following things will happen:

- the pod will be gracefully terminated,

- the ping command that we left running will fail,

- the replica set will notice that it doens't have the right count of pods and 
  create a replacement pod,

- that new pod will have a different IP address (so the `ping` command won't recover).

---

## Assignment 2: first service

1. What command can give our Tomcat server a stable DNS name and IP address?

  (An address that doesn't change when something bad happens to the container.)

2. What commands would you run to connect to Tomcat with that address?

   (Use the `shpod` environment if necessary.)

3. If we delete the pod that holds Tomcat, does the IP address still work?

(answers on next slide)

---

class: answers

## Answers

1. We need to create a *Service* for our deployment, which will have a 
   *ClusterIP* that is usable from within the cluster. One way is
   with `kubectl expose deployment littletomcat --port=8080` 
   (The Tomcat image is listening on port 8080 according to Docker Hub).
   Another way is with `kubectl create service clusterip littletomcat --tcp 8080`

2. In the `shpod` environment that we started earlier:
   ```bash
   # Install curl
   apk add curl
   # Make a request to the littletomcat service
   curl http://littletomcat:8080
   ```

3. Yes. If we delete the pod, another will be created to replace it. 
   The *ClusterIP* will still work.

   (Except during a short period while the replacement container is being started.)
