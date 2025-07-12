# Step-by-Step: Assign External IP to Ingress via MetalLB
## Step 1: Install MetalLB

```
```bash
helm repo add metallb https://metallb.github.io/metallb && helm repo update
helm install metallb metallb/metallb \
  --namespace metallb-system --create-namespace
```

```yaml
# metallb/pool.yaml – advertise the chosen LoadBalancer IP
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: rancher-ip
  namespace: metallb-system
spec:
  addresses:
  - 91.99.154.111-91.99.154.111
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: rancher-ip
  namespace: metallb-system
spec:
  ipAddressPools:
  - rancher-ip
```
Apply it:
```
kubectl apply -f metallb-config.yaml
```
This tells MetalLB: “You’re allowed to assign 91.99.154.111 as an external IP.”