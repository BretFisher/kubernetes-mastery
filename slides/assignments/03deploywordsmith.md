name: assignment3

# Assignment 3: deploy wordsmith

- Let's deploy another application called *wordsmith*

- Wordsmith has 3 components:

  - a web frontend: `bretfisher/wordsmith-web`

  - a API backend: `bretfisher/wordsmith-words` (NOTE: won't run on Raspberry Pi's arm/v7 yet [GH Issue](https://github.com/carlossg/docker-maven/issues/213))

  - a postgres database: `bretfisher/wordsmith-db`

- We have built images for these components, and pushed them on the Docker Hub

- We want to deploy all 3 components on Kubernetes

- We want to be able to connect to the web frontend with our browser

---

## Wordsmith details

- Here are all the network flows in the app:

  - the web frontend listens on port 80

  - the web frontend connects to the API at the address http://words:8080

  - the API backend listens on port 8080

  - the API connects to the database with the connection string pgsql://db:5432

  - the database listens on port 5432

---

## Winning conditions

- After deploying and connecting everything together, open the web frontend

- This is what we should see:

  ![Screen capture of the wordsmith app, with lego bricks showing the text "The nørdic whale smokes the nørdic whale"](images/wordsmith.png)

  (You will probably see a different sentence, though.)

- If you see empty LEGO bricks, something's wrong ...

---

## Scaling things up

- If we reload that page, we get the same sentence

- And that sentence repeats the same adjective and noun anyway

- Can we do better?

- Yes, if we scale up the API backend!

- Try to scale up the API backend and see what happens

.footnote[Wondering what this app is all about?
<br/>
It was a demo app showecased at DockerCon]

---

class: answers

## Answers

First, we need to create deployments for all three components:
```bash
kubectl create deployment db --image=bretfisher/wordsmith-db
kubectl create deployment web --image=bretfisher/wordsmith-web
kubectl create deployment words --image=bretfisher/wordsmith-words
```

Note: we need to use these exact names, because these names will be used for the *service* that we will create and their DNS entries as well. To put it differently: if our code connects to `words` then the service should be named `words` and the deployment should also be named `words` (unless we want to write our own service YAML manifest by hand; but we won't do that yet).

---

class: answers

## Answers

Then, we need to create the services for these deployments:
```bash
kubectl expose deployment db --port=5432
kubectl expose deployment web --port=80 --type=NodePort
kubectl expose deployment words --port=8080
```
or
```bash
kubectl create service clusterip db --tcp=5432
kubectl create service nodeport web --tcp=80
kubectl create service clusterip words --tcp=8080
```

Find out the node port allocated to `web`: `kubectl get service web`

Open it in your browser. If you hit "reload", you always see the same sentence.

---

class: answers

## Answers

Finally, scale up the API for more words on refresh:
```bash
kubectl scale deployment words --replicas=5
```

If you hit "reload", you should now see different sentences each time.
