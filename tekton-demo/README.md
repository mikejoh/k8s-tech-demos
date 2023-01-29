# tekton-demo

## Versions

* `kind` - `v0.17.0`
* Tekton `pipeline` - Latest (installed via `kubectl`)
* `tkn` - `v0.29.0` (Tekton CLI)

## Quick start
1. Create a Tekton demo cluster
```
kind create cluster --config=tekton-cluster.yaml --image=kindest/node:v1.25.3
```

2. Install nginx controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Wait for nginx controller Pod to be `Ready`:
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

3. Install the Tekton CLI `tkn`
```
curl -LO https://github.com/tektoncd/cli/releases/download/v0.29.0/tkn_0.29.0_Linux_x86_64.tar.gz
tar xzvf tkn_0.29.0_Linux_x86_64.tar.gz
sudo mv tkn /usr/local/bin
rm -f tkn_0.29.0_Linux_x86_64.tar.gz
```

4. Install Tekton
```
kubectl apply -f https://storage.googleapis.com/tekton-reqleases/pipeline/latest/release.yaml
```

5. Install the Tekton Dashboard (read-only, see [this](https://tekton.dev/docs/dashboard/install/) link for more info):
```
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

6. Port-forward and browse to the Tekton Dashboard:
```
kubectl port-forward -n tekton-pipelines svc/tekton-dashboard 9097:9097
```
and now browse to: http://localhost:9097

7. Complete the official getting started guides [here](https://tekton.dev/docs/getting-started/)!
