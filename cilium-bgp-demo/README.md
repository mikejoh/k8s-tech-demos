# cilium-bgp-demo

_See the pre-req of the needed tools [here](https://docs.cilium.io/en/v1.12/gettingstarted/kind/)._

_We're using v1.12.2 of Cilium in this demo._

## Demo overview

This demo shows how to use the built-in BGP control plane feature in Cilium, this was added [here](https://github.com/cilium/cilium/pull/18860) in the `v1.12.0` release on the 20:th of July 2022. Today there's a bunch of different guides that you can follow to advertise IP ranges assigned and handled by Cilium:
* Use BIRD
* Use kube-router
* Use the older BGP implementation within Cilium that utilized MetalLB.
* Enable BGP control plane

# TODO: Continue here!
There's no clear road map on where the BGP control plane feature will be at in the future, i

The BGP control plane feature is toggled by adding the following flag to the `cilium` agent: `--enable-bgp-control-plane=true`. What this does under the hood is to start a controller that listens for `CiliumBGPPeeringPolicy` CRDs. The BGP control plane is using the `gobgp` API to handle all BGP logic.

# How-to
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
kind load docker-image quay.io/cilium/cilium:v1.12.2 --name cilium-bgp-cluster
```
4. Install `cilium`:
```
helm install cilium cilium/cilium --version 1.12.2 \
   --namespace kube-system \
   --set image.pullPolicy=IfNotPresent \
   --set ipam.mode=kubernetes \
   --set bgpControlPlane.enabled=true
```
5. Start a `gobgpd` container in the same network as the `kind` cluster created above, default network is `kind`:
```
docker run --network kind --name gobgpd
``` 
