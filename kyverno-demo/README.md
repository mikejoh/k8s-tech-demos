# kyverno-demo

## Quick start

1. Create a `kyverno` cluster:
```
kind create cluster --config=kyverno-cluster.yaml --image=kindest/node:v1.25.3
```

2. Install `kyverno` into the `kyvverno` namespace:
```
helm upgrade \
    --install \
    --atomic \
    --debug \
    --namespace kyverno \
    --create-namespace \
    --version 2.7.0 \
    kyverno \
    kyverno/kyverno \
    -f ./values.yaml
```

3. Install the sample policy:
```
kubectl apply -f policies/require-labels.yaml
```
_This sample policy has a `validation` rule that checks if a certain label is set on Pod resources._

4. Apply the `Pod` manifest (which doesn't have the required label):
```
kubectl apply -f pod.yaml
```
5. Check the Policy reports for the policy:
```
kubectl get policyreports.wgpolicyk8s.io cpol-require-labels
```
This will show you the status of the policy, if you list the Policy reports in e.g. YAML then you'll see more information on the policy evaluation.
6. Change the `validationFailureAction` field of the policy to `Enforce` and re-apply the policy, first delete the Pod:
```
kubectl delete -f pod.yaml
kubectl apply -f policies/require-labels.yaml
```
7. Re-apply the Pod manifest and check the returning error:
```
kubectl apply -f pod.yaml
```

You can also download the `kyverno` CLI tool and run a check using a policy against a resource before you apply the manifest, a good use case for this is as a pre-validation step in CI/CD pipelines.

If you download the CLI you can run the following to see what the result would be using the policy we're using in this demo:
```
kyverno apply policies/require-labels.yaml --resource pod.yaml
```
Which would result in the following error:
```
Applying 1 policy rule to 1 resource...

policy require-labels -> resource default/Pod/nginx failed:
1. check-for-labels: validation error: label app.kubernetes.io/name is required. rule check-for-labels failed at path /metadata/labels/

pass: 0, fail: 1, warn: 0, error: 0, skip: 2
```

To check Policy reports in all namespaces to get an overview of what and where you have resources that violates the policy:
```
kubectl get policyreports.wgpolicyk8s.io -A
```

## Policies

To install the [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) policies you can run:
```
helm install kyverno-policies kyverno/kyverno-policies -n kyverno
```
To see what's included in this Helm chart you can read up on them [here](https://kyverno.io/policies/pod-security/).