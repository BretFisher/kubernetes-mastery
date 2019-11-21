# Authoring YAML

- There are various ways to *generate* YAML with Kubernetes, e.g.:

  - `kubectl run`

  - `kubectl create deployment` (and a few other `kubectl create` variants)

  - `kubectl expose`

--

- These commands use "generators" because the API only accepts YAML (actually JSON) 

--

- Pro: They are easy to use

- Con: They have limits

--

- When and why do we need to write our own YAML?

- How do we write YAML from scratch?

---

## The limits of generated YAML

- Many advanced (and even not-so-advanced) features require to write YAML:

  - pods with multiple containers

  - resource limits

  - healthchecks

  - many other resource options

--

- Many other resource types don't have their own commands!

  - DaemonSets
  
  - StatefulSets

  - and more!

- How do we access these features?

---

## We don't have to start from scratch

- Output YAML from existing resources

  - Create a resource (e.g. Deployment)

  - Dump its YAML with `kubectl get -o yaml ...`

  - Edit the YAML

  - Use `kubectl apply -f ...` with the YAML file to:

    - update the resource (if it's the same kind)

    - create a new resource (if it's a different kind)

--

- Or... we have the docs, with good starter YAML

  - [StatefulSet](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#creating-a-statefulset), 
[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#create-a-daemonset), 
[ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-a-configmap),
[and a ton more on GitHub](https://github.com/kubernetes/website/tree/master/content/en/examples)

--

- Or... we can use `-o yaml --dry-run`

---

## Generating YAML without creating resources

- We can use the `-o yaml --dry-run` option combo with `run` and `create`

.exercise[

- Generate the YAML for a Deployment without creating it:
  ```bash
  kubectl create deployment web --image nginx -o yaml --dry-run
  ```

- Generate the YAML for a Namespace without creating it:
  ```bash
  kubectl create namespace awesome-app -o yaml --dry-run
  ```
]

- We can clean up the YAML even more if we want

  (for instance, we can remove the `creationTimestamp` and empty dicts)

---

## Try `-o yaml --dry-run` with other create commands

```bash
  clusterrole         # Create a ClusterRole.
  clusterrolebinding  # Create a ClusterRoleBinding for a particular ClusterRole.
  configmap           # Create a configmap from a local file, directory or literal.
  cronjob             # Create a cronjob with the specified name.
  deployment          # Create a deployment with the specified name.
  job                 # Create a job with the specified name.
  namespace           # Create a namespace with the specified name.
  poddisruptionbudget # Create a pod disruption budget with the specified name.
  priorityclass       # Create a priorityclass with the specified name.
  quota               # Create a quota with the specified name.
  role                # Create a role with single rule.
  rolebinding         # Create a RoleBinding for a particular Role or ClusterRole.
  secret              # Create a secret using specified subcommand.
  service             # Create a service using specified subcommand.
  serviceaccount      # Create a service account with the specified name.
```

- Ensure you use valid `create` commands with required options for each

---

## Writing YAML from scratch, AKA "YAML The Hard Way"

- Paying homage to Kelsey Hightower's "[Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)"

--

- A reminder about manifests: 

  - Each file contains one or more manifests

  - Each manifest describes an API object (deployment, service, etc.)

  - Each manifest needs four parts (root key:values in the file)


```yaml
    apiVersion:  # find with "kubectl api-versions"
    kind:        # find with "kubectl api-resources"
    metadata:
    spec:        # find with "kubectl describe pod"
```

--

- Those three `kubectl` commands, plus the API docs, is all we'll need

---

## General workflow of YAML from scratch

- Find the resource `kind` you want to create (`api-resources`)

--

- Find the latest `apiVersion` your cluster supports for `kind` (`api-versions`)

--

- Give it a `name` in metadata (minimum)

--

- Dive into the `spec` of that `kind` 
  - `kubectl explain <kind>.spec`
  - `kubectl explain <kind> --recursive`

--

- Browse the docs [API Reference](https://kubernetes.io/docs/reference/) for your cluster version to supplement

--

- Use `--dry-run` and `--server-dry-run` for testing

- `kubectl create` and `delete` until you get it right


<!--TODO: create example of YAML from scratch  -->
---

## Advantage of YAML

- Using YAML (instead of `kubectl run`/`create`/etc.) allows to be *declarative*

- The YAML describes the desired state of our cluster and applications

- YAML can be stored, versioned, archived (e.g. in git repositories)

- To change resources, change the YAML files

  (instead of using `kubectl edit`/`scale`/`label`/etc.)

- Changes can be reviewed before being applied

  (with code reviews, pull requests ...)

- This workflow is sometimes called "GitOps"

  (there are tools like Weave Flux or GitKube to facilitate it)

---

## YAML in practice

- Get started with `kubectl run`/`create`/`expose`/etc.

- Dump the YAML with `kubectl get -o yaml`

- Tweak that YAML and `kubectl apply` it back

- Store that YAML for reference (for further deployments)

- Feel free to clean up the YAML:

  - remove fields you don't know

  - check that it still works!

- That YAML will be useful later when using e.g. Kustomize or Helm
