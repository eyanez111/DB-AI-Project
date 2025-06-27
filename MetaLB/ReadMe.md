# Step-by-Step: Assign External IP to Ingress via MetalLB
## Step 1: Install MetalLB

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
Wait a few seconds for the MetalLB pods to start:
```
```
kubectl get pods -n metallb-system
```

# tep 2: Create an IP Address Pool
Create a ConfigMap that tells MetalLB to use your public IP (91.99.154.111):

# metallb-config.yaml

```
Version: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: public-ip-pool
spec:
  addresses:
  - 91.99.154.111/32
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  namespace: metallb-system
  name: advert
```

Apply it:
```
kubectl apply -f metallb-config.yaml
```
This tells MetalLB: “You’re allowed to assign 91.99.154.111 as an external IP.”