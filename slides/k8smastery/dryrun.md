# Using server-dry-run and diff

- We already talked about using `--dry-run` for building YAML

- Let's talk more about options for testing YAML

- Including testing against the live cluster API!

---

## Using `--dry-run` with `kubectl apply`

- The `--dry-run` option can also be used with `kubectl apply`

- However, it can be misleading (it doesn't do a "real" dry run)

- Let's see what happens in the following scenario:

  - generate the YAML for a Deployment

  - tweak the YAML to transform it into a DaemonSet

  - apply that YAML to see what would actually be created

---

## The limits of `kubectl apply --dry-run`

.exercise[

- Generate the YAML for a deployment:
  ```bash
  kubectl create deployment web --image=nginx -o yaml > web.yaml
  ```

- Change the `kind` in the YAML to make it a `DaemonSet`

- Ask `kubectl` what would be applied:
  ```bash
  kubectl apply -f web.yaml --dry-run --validate=false -o yaml
  ```

]

The resulting YAML doesn't represent a valid DaemonSet.

---

## Server-side dry run

- Since Kubernetes 1.13, we can use [server-side dry run and diffs](https://kubernetes.io/blog/2019/01/14/apiserver-dry-run-and-kubectl-diff/)

- Server-side dry run will do all the work, but *not* persist to etcd

  (all validation and mutation hooks will be executed)

.exercise[

- Try the same YAML file as earlier, with server-side dry run:
  ```bash
  kubectl apply -f web.yaml --server-dry-run --validate=false -o yaml
  ```

]

The resulting YAML doesn't have the `replicas` field anymore.

Instead, it has the fields expected in a DaemonSet.

---

## Advantages of server-side dry run

- The YAML is verified much more extensively

- The only step that is skipped is "write to etcd"

- YAML that passes server-side dry run *should* apply successfully

  (unless the cluster state changes by the time the YAML is actually applied)

- Validating or mutating hooks that have side effects can also be an issue

---

## `kubectl diff`

- Kubernetes 1.13 also introduced `kubectl diff`

- `kubectl diff` does a server-side dry run, *and* shows differences

.exercise[

- Try `kubectl diff` on a simple Pod YAML:
  ```bash
  curl -O https://k8smastery.com/just-a-pod.yaml
  kubectl apply -f just-a-pod.yaml
  # edit the image tag to :1.17
  kubectl diff -f just-a-pod.yaml
  ```

]

Note: we don't need to specify `--validate=false` here.

