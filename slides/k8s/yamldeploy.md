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

## Basic parts of any Kubernetes resource in YAML

- Can be in YAML or JSON, but YAML is 💯

--

- Each file contains one or more manifests

--

- Each manifest describes an API object (deployment, service, etc.)

--

- Each manifest needs four parts (root key:values in the file)


```yaml
apiVersion:  # find with "kubectl api-versions"
kind:        # find with "kubectl api-resources"
metadata:
spec:        # find with "kubectl describe pod"
```

--

- We'll learn later how to build YAML from scratch

---

## A simple Pod in YAML

- This is a single manifest that creates one Pod

```yaml
apiVersion: v1   
kind: Pod        
metadata:
  name: nginx
spec:            
  containers:
  - name: nginx
    image: nginx:1.17.3
```

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

## Deployment and Service manifests in one YAML file

.small[
```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: mynginx
  spec:
    type: NodePort
    ports:
    - port: 80
    selector:
      app: mynginx
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mynginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: mynginx
    template:
      metadata:
        labels:
          app: mynginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.17.3
```
]

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

## Deploying DockerCoins with YAML

- We provide a YAML manifest with all the resources for DockerCoins

  (Deployments and Services)

- We can use it if we need to deploy or redeploy DockerCoins

.exercise[

- Deploy or redeploy DockerCoins:
  ```bash
  kubectl apply -f https://k8smastery.com/dockercoins.yaml
  ```

]

--

- Note the warnings if you already had the resources created

- This is because we didn't use `apply` before

- This is OK for us learning, so ignore the warnings

- Generally in production you want to stick with one method or the other

