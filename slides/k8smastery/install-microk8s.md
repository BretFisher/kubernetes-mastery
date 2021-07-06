name: microk8s

# MicroK8s (Linux)

- [Easy install](https://microk8s.io/) and management of local Kubernetes

--

- Made by Canonical (Ubuntu). Installs using `snap`. Works nearly everywhere
- Has lots of other features with its `microk8s` CLI

--

- But, requires you [install `snap`](https://snapcraft.io/docs/installing-snapd) if not on Ubuntu
- Runs on containerd rather than Docker, no biggie
- Needs alias setup for `microk8s kubectl`

--

.exercise[

- Install `microk8s`,change group permissions, then set alias in bashrc
  ``` bash
  sudo snap install microk8s --classic
  sudo usermod -a -G microk8s <username>
  echo "alias kubectl='microk8s kubectl'" >> ~/.bashrc
  # log out and back in if using a non-root user
  ```

]

---

## MicroK8s Additional Info

- We'll need these later (these are done for us in Docker Desktop and minikube):

.exercise[

- Create kubectl config file
  ``` bash
  microk8s kubectl config view --raw > $HOME/.kube/config
  ```

- Install CoreDNS in Kubernetes
  ``` bash
  sudo microk8s enable dns
  ```
]

- You can also install other plugins this way like `microk8s enable dashboard` or `microk8s enable ingress`

---

## MicroK8s Troubleshooting

- Run a check for any config problems

.exercise[

- Test MicroK8s config for any potental problems
  ``` bash
  sudo microk8s inspect
  ```
]

- If you also have Docker installed, you can ignore warnings about iptables and registries

- See [troubleshooting site](https://microk8s.io/docs/troubleshooting) if you have issues
