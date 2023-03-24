# argocd-demo

Components:
* `kind`
* `jq`

## Links

* https://argocd-applicationset.readthedocs.io/en/stable/Use-Cases/#use-case-cluster-add-ons
* https://github.com/argoproj/applicationset
* https://foxutech.medium.com/argo-cd-sync-policies-and-options-61e88c5a02da
* https://www.arthurkoziel.com/fixing-argocd-crd-too-long-error/

## Pre-req

1. Start clusters:
```
kind create cluster --config=argocd-cluster.yaml
kind create cluster --config=downstream-cluster.yaml
```

2. Install ArgoCD, change context `argocd-cluster`:
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. Install `argocd` CLI.

4. Login to ArgoCD UI:
```
argocd admin initial-password -n argocd
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Browse to `http://localhost:8080`.

5. Add the downstream cluster as a cluster in ArgoCD:
_We need to change `https://127.0.0.1` to the IP address of the Kubernetes endpoint, this address is internal in the network bridge where both `kind` clusters are connected to._
```
kubectl config set-cluster kind-downstream-cluster --server https://$(kubectl get endpoints kubernetes -o json | jq -r '.subsets[].addresses[].ip'):6443

argocd cluster add kind-downstream-cluster
```