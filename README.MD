## Setup

To reproduce the example project please walk through the following steps:

- [Setup](#setup)
  - [Fork and clone this example](#fork-and-clone-this-example)
  - [Create local k8s kind cluster](#create-local-k8s-kind-cluster)
  - [Setup Flux](#setup-flux)
- [Cleanup](#cleanup)
### Fork and clone this example

Create a fork of this repository, clone it to your local machine and change into its directory.
The following is expected to be executed within there.

### Create local k8s kind cluster

[Install k8s kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) or use the Kubernetes distribution you prefer.

```shell
# Create new cluster and configure kubectl
# depending on your system setup you may need to run the kind commands als privileged user or with `sudo`
kind create cluster --config ./_kind/config.yaml --image=kindest/node:v1.24.0
kind export kubeconfig --kubeconfig $HOME/.kube/config

### Output will be something like:
# Creating cluster "kind" ...
#  ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
#  ✓ Preparing nodes 📦
#  ✓ Writing configuration 📜
#  ✓ Starting control-plane 🕹️
#  ✓ Installing CNI 🔌
#  ✓ Installing StorageClass 💾
# Set kubectl context to "kind-kind"
# You can now use your cluster with:
#
# kubectl cluster-info --context kind-kind
#
# Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
# Set kubectl context to "kind-kind"
```

### Setup Flux

[Install flux cli](https://fluxcd.io/docs/cmd/#install-using-bash) for bootstrapping FluxCD, the GitOps operator synchronizing the K8s manifests into your cluster.

```shell
# Initialize the cluster and configure synchronization from git repository
export GITHUB_TOKEN= # your github personal access token with repository scope
export REPO_OWNER=heubeck # replace with your github account
export REPO_NAME=cryostat-playground
flux bootstrap github \
  --owner=$REPO_OWNER \
  --repository=$REPO_NAME \
  --private=false \
  --personal=true \
  --path=bootstrap \
  --branch=main

### Output (shrinked):
# ► connecting to github.com
# ► cloning branch "main" from Git repository "https://github.com/heubeck/cloudland-canary.git"
# ✔ cloned repository
# ► generating component manifests
# ✔ generated component manifests
# ✔ component manifests are up to date
# ► installing components in "flux-system" namespace
# ✔ installed components
# ✔ reconciled components
#  ...
# ◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
# ✔ Kustomization reconciled successfully
# ► confirming components are healthy
# ✔ helm-controller: deployment ready
# ✔ kustomize-controller: deployment ready
# ✔ notification-controller: deployment ready
# ✔ source-controller: deployment ready
# ✔ all components are healthy
```

The bootstrap command may vary on a different constellation, like with an organization or non-public repostory. Please check the Flux documentation.

It will last some minutes for the cluster components to become ready. You can check for completion and validate the setup by looking into Flux' Kustomization resources:

```shell
# We are ready to go as soon as all listed kustomization resources are `READY`
kubectl get kustomization -A

### Output as soon as everything is ready:
# NAMESPACE     NAME                     AGE     READY   STATUS
# flux-system   apps                     3m31s   True    Applied revision: main/71a913a6c2020e976917eecfc3267405a610410e
# flux-system   flux-system              3m58s   True    Applied revision: main/71a913a6c2020e976917eecfc3267405a610410e
# flux-system   infrastructure-layer-1   3m31s   True    Applied revision: main/71a913a6c2020e976917eecfc3267405a610410e
# flux-system   infrastructure-layer-2   3m31s   True    Applied revision: main/71a913a6c2020e976917eecfc3267405a610410e
```

## Cleanup

```shell
# Stop and remove the kind cluster
kind delete cluster
```

---
