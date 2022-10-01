# cilium-bgp-demo

_See the pre-req of the needed tools [here](https://docs.cilium.io/en/v1.12/gettingstarted/kind/)._

_We're using v1.12.2 of Cilium in this demo._

## Quick start
1. Create cluster
```
kind create cluster --config=cilium-bgp-cluster.yaml
```
_Note that the `kind` default subnets are: `10.244.0.0/16` (Pod CIDR) and `10.96.0.0/12` (Service CIDR)_

2. Add the Cilium `helm` chart repo:
```
helm repo add cilium https://helm.cilium.io/
helm repo update
```
3. Pre-load the `cilium` images into the `kind` cluster nodes:
```
docker pull quay.io/cilium/cilium:v1.12.2
kind load docker-image quay.io/cilium/cilium:v1.12.2
```
4. Install `cilium`:
```
helm install cilium cilium/cilium --version 1.12.2 \
   --namespace kube-system \
   --set image.pullPolicy=IfNotPresent \
   --set ipam.mode=kubernetes \
   --set bgpControlPlane.enabled=true
```