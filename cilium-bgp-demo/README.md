# cilium-bgp-demo

_See the pre-req of the needed tools [here](https://docs.cilium.io/en/stable/installation/kind/#install-dependencies)._

## Demo overview

_This demo shows how to use the built-in BGP control plane feature in Cilium, which was added [here](https://github.com/cilium/cilium/pull/18860) in the `v1.12.0` release on the 20:th of July 2022. Today there's a bunch of different official guides that you can follow to advertise IP ranges assigned and handled by Cilium._

The BGP control plane feature is toggled by adding the following flag to the `cilium-agent`: `--enable-bgp-control-plane=true`. The BGP control plane is using the Go implementation of BGP, [`GoBGP`](https://github.com/osrg/gobgp). The `GoBGP` project provides both a server and a client side binary to start a BGP service and to be able to query it. Similar to `bird` and `birdc`. The key difference is that `GoBGP` has a gRPC API that we can interact with.

One of the reasons i wanted to run `gobgpd` was simple because it's what drives the BGP implementation within Cilium. It's basically a good way of familiarizing myself with whats going on under the hood.

## How-to

1. Create the `kind` cluster:

```
kind create cluster --config=cilium-bgp-cluster.yaml --name cilium-bgp
```

_Note that the `kind` default subnets are: `10.244.0.0/16` (Pod CIDR) and `10.96.0.0/12` (Service CIDR)_

2. Add the Cilium `helm` chart repo:

```
helm repo add cilium https://helm.cilium.io/
helm repo update
```

3. Pre-load the `cilium` images into the `kind` cluster nodes:

```
docker pull quay.io/cilium/cilium:v1.16.0
kind load docker-image quay.io/cilium/cilium:v1.16.0 --name cilium-bgp
```

4. Install `cilium`:

```
helm upgrade --install cilium cilium/cilium --version 1.16.0 \
   --namespace kube-system \
   --set image.pullPolicy=IfNotPresent \
   --set ipam.mode=kubernetes \
   --set bgpControlPlane.enabled=true
```

5. Build the `gobgpd` (running `v3.30.0` of `GoBGP`) Docker image, using the provided `Dockerfile`:

```
docker build . -t gobgpd:v3.30.0
```

6. Start a `gobgpd` container in the same network as the `kind` cluster created above, default network is `kind`. We'll use a configuration that is easy to understand and follow, we'll also expose the gRPC port to be able to use the `gobgp` CLI to interact with the `gobgpd` process:

```
docker run --rm --network kind -v $(pwd)/bgp-conf.yaml:/bgp-conf.yaml -p 50051:50051 --name gobgpd -it gobgpd:v3.30.0
```

_Note that the container is started in interactive mode so you'll see the logs from `gobgpd`. The log level is `debug`._

7. Apply the Cilium BGP peering policy and check the logs of `cilium-agent` and `gobgpd`. The nodes shall start to peer with `gobgpd` and announce their PodCIDRs. To verify that this has happened use the `gobgp` CLI tool to check the status of neighbors aswell as the routing information:

```
kubectl apply -f cilium-bgp-configuration.yaml
```

To check the status of `cilium-agent` BGP sessions you can do:

```
for pod in $(kubectl get pods -n kube-system -l app.kubernetes.io/name=cilium-agent -o name)
do
  kubectl -n kube-system exec $pod -c cilium-agent -- cilium bgp peers
  kubectl -n kube-system exec $pod -c cilium-agent -- cilium bgp routes advertised
done
```

_`gobgpd` exposes the gRPC endpoint is on `0.0.0.0:50051`, use the `gobgp` binary to query the API._

## Monitoring

1. Deploy `kube-prometheus-stack`, a bit overkill for a `kind` cluster but it will give you a head start:

```
TODO add stuff here!
```

1. Enable and deploy relevant monitoring in Cilium:

```
helm upgrade --install cilium cilium/cilium --version 1.16.0 \
   --namespace kube-system \
   --set image.pullPolicy=IfNotPresent \
   --set ipam.mode=kubernetes \
   --set bgpControlPlane.enabled=true \
   --set operator.prometheus.enabled=true \
   --set operator.grafana.dashboards.enabled=true
```

## Try various failure scenarios

1. Rollout restart of Cilium, how does that affect the BGP sessions and what can we tweak in terms of timers?
