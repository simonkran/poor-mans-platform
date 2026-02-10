# Poor Man's Platform

A lightweight internal developer platform built on Kubernetes using [kro](https://kro.run), [Flux](https://fluxcd.io), and [Crossplane](https://crossplane.io).

## What it does

Provides three custom resources:

- **Workspace** - Creates a GitHub repo (from template), namespace, and Flux GitOps sync
- **Work** - Deploys a containerized workload (Deployment + Service)
- **SQLDatabase** - Provisions a MariaDB instance via Helm

## Setup

```shell
kind create cluster --name pmp

# Flux
helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator \
  --namespace flux-system --create-namespace
kubectl apply -k flux/

# kro
helm install kro oci://registry.k8s.io/kro/charts/kro \
  --namespace kro-system --create-namespace

# Crossplane
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update crossplane-stable
helm install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system --create-namespace

# GitHub credentials for Crossplane
kubectl create secret generic github-credentials \
  --namespace crossplane-system \
  --from-literal=credentials='{"token":"<YOUR_GITHUB_PAT>","owner":"<YOUR_GITHUB_USERNAME>"}'

# Apply providers and resource definitions
kubectl apply -k crossplane/
kubectl apply -k kro/
```

## Usage

Create a workspace:

```yaml
apiVersion: v1alpha1
kind: Workspace
metadata:
  name: my-workspace
spec:
  name: my-workspace
```

Deploy a workload:

```yaml
apiVersion: v1alpha1
kind: Work
metadata:
  name: api
spec:
  name: api
  workspace: my-workspace
  image: nginx:latest
```

Add a database:

```yaml
apiVersion: v1alpha1
kind: SQLDatabase
metadata:
  name: mydb
spec:
  name: mydb
  workspace: my-workspace
```
