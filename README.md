# Poor Man's Platform

```shell
kind create cluster --name pmp

helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator \
  --namespace flux-system \
  --create-namespace

kubectl apply -k flux/ 

helm install kro oci://registry.k8s.io/kro/charts/kro \
  --namespace kro-system \
  --create-namespace

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update crossplane-stable
helm install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace


kubectl create secret generic github-credentials \
  --namespace crossplane-system \
  --from-literal=credentials='{"token":"<YOUR_GITHUB_PAT>","owner":"<YOUR_GITHUB_USERNAME>"}'

kubectl apply -k crossplane/

kubectl apply -k kro/
```
