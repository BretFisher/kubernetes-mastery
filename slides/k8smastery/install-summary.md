name: install

# Getting a Kubernetes cluster for learning

- Best: Get a environment locally

  - Docker Desktop (Win/macOS), minikube (Win Home), or microk8s (Linux)
  - Small setup effort; free; flexible environments
  - Requires 2GB+ of memory

--

- Good: Setup a cloud Linux host to run microk8s

  - Great if you don't have the local resources to run Kubernetes
  - Small setup effort; only free for a while
  - My $50 DigitalOcean coupon lets you run Kubernetes free for a month

--

- Last choice: Use a browser-based solution

  - Low setup effort; but host is short-lived and has limited resources
  - Not all hands-on examples will work in the browser sandbox

--

- For all environments, we'll use `shpod` container for tools
