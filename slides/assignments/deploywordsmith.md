# Assignment: deploy wordsmith

- Let's deploy another application called *wordsmith*

- Wordsmith has 3 components:

  - a web frontend

  - an API backend

  - a database

- We have built images for these components, and pushed them on the Docker Hub

- We want to deploy all 3 components on Kubernetes

- We want to be able to connect to the web frontend with our browser

---

## Wordsmith details

- Here are the names of the images that we've prepared:

  - `jpetazzo/wordsmith-web:latest` for the web frontend

  - `jpetazzo/wordsmith-words:latest` for the API

  - `jpetazzo/wordsmith-db:latest` for the database

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

- Yes, there is some repetition in that sentence; that's OK for now

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
It was a demo app showecased at DockerCon Europe 2017 in Copenhagen.]

---

class: answers

## Answers

First, we need to create deployments for all three components:
```bash
kubectl create deployment db --image=jpetazzo/wordsmith-db
kubectl create deployment web --image=jpetazzo/wordsmith-web
kubectl create deployment words --image=jpetazzo/wordsmith-words
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

Find out the node port allocated to `web`:
```bash
kubectl get service web
```

Open it in your browser.

If you hit "reload", you should always see the same sentence, however.

---

class: answers

## Answers

Finally, scale up the API:
```bash
kubectl scale deployment words --replicas=9
```

If you hit "reload", you should now see different sentences each time.
