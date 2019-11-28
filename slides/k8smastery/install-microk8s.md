# MicroK8s (Linux)

- [Easy install](https://microk8s.io/) and management of local Kubernetes

--

- Made by Canonical (Ubuntu). Installs using `snap`. Works nearly everywhere
- Has lots of other features with its `microk8s.` CLI

--

- But, requires you [install `snap`](https://snapcraft.io/docs/installing-snapd) if not on Ubuntu
- Runs on containerd rather than Docker, no biggie
- Needs alias setup for `microk8s.kubectl`

--

.exercise[

- Install `microk8s`,change group permissions, then set alias in bashrc
  ``` bash
  sudo snap install microk8s --classic
  sudo usermod -a -G microk8s <username>
  echo "alias kubectl='microk8s.kubectl'" >> ~/.bashrc
  ```

]
