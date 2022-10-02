# cilium-bgp-demo

_See the pre-req of the needed tools [here](https://docs.cilium.io/en/v1.12/gettingstarted/kind/)._

_We're using v1.12.2 of Cilium in this demo._

## Demo overview

This demo shows how to use the built-in BGP control plane feature in Cilium, which was added [here](https://github.com/cilium/cilium/pull/18860) in the `v1.12.0` release on the 20:th of July 2022. Today there's a bunch of different official guides that you can follow to advertise IP ranges assigned and handled by Cilium:
* Via BIRD
* Via kube-router
* Use the older BGP implementation within Cilium that utilized MetalLB.
* Enable the BGP control plane feature

At the moment there's no clear road map in regards to the BGP control plane from the Cilium project. The current abstraction is the `CiliumBGPPeeringPolicy` which has the bare minimum to create one or more BGP peers. It doesn't provide the same feature set (BGP wise) that e.g. Calico does with BIRD.

There's a couple of CFPs (Cilium Feature Proposals) that might end up in Cilium and extended the BGP support to e.g. more advanced use-cases such as Route Reflector configurations and password support when peering.

The BGP control plane feature is toggled by adding the following flag to the `cilium` agent: `--enable-bgp-control-plane=true`. What this does under the hood is to start a controller that listens for `CiliumBGPPeeringPolicy` CRDs. The BGP control plane is using the Go implementation of BGP, [`GoBGP`](https://github.com/osrg/gobgp). The `GoBGP` project provides both a server and a client side binary to start a BGP service and to be able to query it. Similar to `bird` and `birdc`. The key difference is that `GoBGP` has a gRPC API that we can interact with.

One of the reasons i wanted to run `gobgpd` was simple because it's what drives the BGP implementation within Cilium. It's basically a good way of familiarizing myself with whats going on under the hood.

# How-to
1. Create the `kind` cluster:
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
5. Build the `gobgpd` (running `v3.7.0` of `GoBGP`) Docker image, using the provided `Dockerfile`:
```
docker build . -t gobgpd:v3.7.0
```
6. Start a `gobgpd` container in the same network as the `kind` cluster created above, default network is `kind`. We'll use a configuration that is easy to understand and follow, we'll also expose the gRPC port to be able to use the `gobgp` CLI to interact with the `gobgpd` process:
```
docker run --rm --network kind -v $(pwd)/bgp-conf.yaml:/bgp-conf.yaml -p 50051:50051 --name gobgpd -it gobgpd:v3.7.0
``` 
_Note that the container is started in interactive mode so you'll see the logs from `gobgpd`. The log level is `debug`._
7. Apply the Cilium BGP peering policy and check the logs of `cilium-agent` and `gobgpd`. The nodes shall start to peer with `gobgpd` and announce their PodCIDRs. To verify that this has happened use the `gobgp` CLI tool to check the status of neighbors aswell as the routing information:
```
kubectl apply -f cilium-bgp-peering-policy.yaml
```
```
gobgp neighbor
gobgp global rib
```

_You'll run `gobgp` locally since we've exposed the gRPC port, it assumes that the gRPC endpoint is at 0.0.0.0:50051_

### TODO
* Make the BGP configuration a bit more predictable with all the Docker network assigned IP addresses. Right now i've assumed a bunch of things in the config.
* Investigate if there's a possibility to increase the Cilium BGP reconcile loop to retry against disconnected peers more often.
