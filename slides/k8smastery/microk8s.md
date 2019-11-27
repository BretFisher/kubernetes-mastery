## microk8s (Linux)

- [Easy install](https://microk8s.io/) and management of local Kubernetes

--

- Made by Canonical (Ubuntu). Installs using `snap`. Works nearly everywhere
- Has lots of other features with its `microk8s.` CLI

--

- But, requires you [install `snap`](https://snapcraft.io/docs/installing-snapd) on non-Ubuntu distros
- Doesn't run on Docker, runs on containerd, no biggie
- Needs alias setup for `microk8s.kubectl`

--

.exercise[

- Install `microk8s`, check status, then set alias
  ``` bash
  sudo snap install microk8s --classic
  sudo microk8s.status --wait-ready
  alias kubectl='microk8s.kubectl'
  ```

]

.footnote[.small[
    Save alias to your `.bashrc`
]]