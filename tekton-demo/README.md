# tekton-demo

## Quick start
1. Create cluster
```
kind create cluster --config=tekton-cluster --image=kindest/node:v1.22.5
```
2. Install nginx controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Wait for nginx controller Pod to be Ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```
3. Install the Tekton CLI `tkn`
```
curl -LO https://github.com/tektoncd/cli/releases/download/v0.22.0/tkn_0.22.0_Linux_x86_64.tar.gz
tar xzvf tkn_0.22.0_Linux_x86_64.tar.gz
sudo mv tkn /usr/local/bin
```
4. Install Tekton
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

## Create and start a `Task`:
1. Create the demo hello world task:
```
kubectl apply -f task-hello.yaml
```
2. Start the task:
```
tkn task start hello
```
3. Display the last run `Task` logs:
```
tkn taskrun logs --last -f
```
This should output something like:
```
[hello] Hello World!
```