# Deploying with YAML

- So far, we created resources with the following commands:

  - `kubectl run`

  - `kubectl create deployment`

  - `kubectl expose`

- We can also create resources directly with YAML manifests

---

## `kubectl apply` vs `create`

- `kubectl create -f whatever.yaml`

  - creates resources if they don't exist

  - if resources already exist, don't alter them
    <br/>(and display error message)

- `kubectl apply -f whatever.yaml`

  - creates resources if they don't exist

  - if resources already exist, update them
    <br/>(to match the definition provided by the YAML file)

  - stores the manifest as an *annotation* in the resource

---

## Creating multiple resources

- The manifest can contain multiple resources separated by `---`

```yaml
 kind: ...
 apiVersion: ...
 metadata:
   name: ...
   ...
 spec:
   ...
 ---
 kind: ...
 apiVersion: ...
 metadata:
   name: ...
   ...
 spec: 
   ...
```

---

## Creating multiple resources

- The manifest can also contain a list of resources

```yaml
 apiVersion: v1
 kind: List
 items:
 - kind: ...
   apiVersion: ...
   ...
 - kind: ...
   apiVersion: ...
   ...
```

---

name: dockercoins

## Deploying DockerCoins with YAML

- Here's a YAML manifest with all the resources for DockerCoins

  (Deployments and Services)

- We can use it if we need to deploy or redeploy DockerCoins

- Yes YAML file commands can use URL's!

.exercise[

- Deploy or redeploy DockerCoins:
  ```bash
  kubectl apply -f https://k8smastery.com/dockercoins.yaml
  ```

]

---

## `Apply` errors for `create` or `run` resources

- Note the warnings if you already had the resources created

- This is because we didn't use `apply` before

- This is OK for us learning, so ignore the warnings

- Generally in production you want to stick with one method or the other

