# Testing ArgoCD locally on `kind`

This guide deploys two small Kubernetes clusters, one where ArgoCD will run and one downstream cluster.

1. Start clusters:
```
kind create cluster --config=argocd-cluster.yaml
kind create cluster --config=downstream-cluster.yaml
```

2. Install ArgoCD (includes all components, otherwise see the `core` version of ArgoCD):
```
kubectl config use-context kind-argocd-cluster
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. Install the `argocd` CLI using `yay`, `brew` or similar.

4. Print the temporary admin password and use that with the username `admin` to login:
```
argocd admin initial-password -n argocd
```

5. Login to the ArgoCD UI:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Browse to `http://localhost:8080`.

6. Login, using the `argocd` CLI, to the local ArgoCD cluster (make sure that the port-forward is active):
```
argocd login localhost:8080
```
Use `admin` and the password from step 5.

7. Add the downstream cluster as a cluster in ArgoCD:

_We need to change `https://127.0.0.1:6443` API server URL in the kubeconfig context since this is used to configure and add the cluster, when ArgoCD tries to communicate with `127.0.0.1` this will fail. The IP address we want to use in this case is the one assign to the Kubernetes `Endpoint` within the downstream cluster. This IP is reachable since both clusters are deployed in the same network._
```
kubectl config use-context kind-downstream-cluster
kubectl config set-cluster kind-downstream-cluster --server https://$(kubectl get endpoints kubernetes -o json | jq -r '.subsets[].addresses[].ip'):6443
```

Now run:
```
kubectl config use-context kind-argocd-cluster
argocd cluster add kind-downstream-cluster
``` 
to add the cluster using details from the `kind-downstream-cluster` context in your local kubeconfig. Browse to https://localhost:8080/settings/clusters to see the newly added cluster.

8. Deploy the provided `ApplicationSet`:
```
kubectl config use-context kind-argocd-cluster
kubectl apply -f k8s-addons-applicationset.yaml
```