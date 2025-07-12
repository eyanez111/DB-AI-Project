# Social Trend Intelligence Platform – Infrastructure Reference Guide (HTTP‑01 with Static IP)

---

## 1  (OPTIONAL) Install MetalLB †


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
  name: public‑ip
  namespace: metallb-system
spec:
  addresses:
  - 91.99.154.111-91.99.154.111  # single IP already routed via A‑records
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: public‑ip
  namespace: metallb-system
spec:
  ipAddressPools:
  - public‑ip
```

#### Picking an IP for MetalLB
````
```bash
kubectl apply -f metallb-config.yaml
````

---

## 2  Install Ingress‑NGINX

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.service.loadBalancerIP=91.99.154.111 \
  --set controller.publishService.enabled=true
```

The controller’s `LoadBalancer` service should now show the external IP `91.99.154.111`.

---

## 3  Install cert‑manager (v1.18.1)

```bash
helm repo add jetstack https://charts.jetstack.io && helm repo update
```
```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --version v1.18.1 \
  --set installCRDs=true
```
```yaml 
kubectl get pods -n cert-manager  # wait until Ready
```

---

## 4  Create the HTTP‑01 ClusterIssuer

```yaml
# cert-manager/clusterissuer-http.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http
spec:
  acme:
    email: admin@${DOMAIN}
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-http-key
    solvers:
    - http01:
        ingress:
          class: nginx   # MUST match the class used in step 2
```

```bash
kubectl apply -f cert-manager/clusterissuer-http.yaml
```

---

## 5  Request Certificates for Each Host

*Let’s Encrypt only allows wildcard certificates via DNS‑01, so we generate one cert per sub‑domain.*

```yaml
# cert-manager/certs.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: rancher-480-io
  namespace: cert-manager
spec:
  secretName: rancher-480-io-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: rancher.${DOMAIN}
  dnsNames:
  - rancher.${DOMAIN}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ai-480-io
  namespace: cert-manager
spec:
  secretName: ai-480-io-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: ai.${DOMAIN}
  dnsNames:
  - ai.${DOMAIN}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: monitoring-480-io
  namespace: cert-manager
spec:
  secretName: monitoring-480-io-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: monitoring.${DOMAIN}
  dnsNames:
  - monitoring.${DOMAIN}
```

```bash
kubectl apply -f cert-manager/certs.yaml
kubectl get certificaterequest -A   # should progress from Pending → Ready in ~1 min
```

> **Path precedence tip** – your Rancher ingress will still claim `/`, but NGINX always picks the most specific prefix. The challenge path `/.well-known/acme-challenge/*` therefore routes correctly to the solver pod created by cert‑manager. No extra annotations are required.

---

## 6  Deploy Ingress Resources

Create minimal ingresses that reference the TLS secrets produced above—**no TLS until certs are Ready**:

```yaml
# ingress/rancher.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rancher-ingress
  namespace: cattle-system
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-http
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - rancher.${DOMAIN}
    secretName: rancher-480-io-tls
  rules:
  - host: rancher.${DOMAIN}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rancher
            port:
              number: 443
---
# Repeat similarly for ai.${DOMAIN} and monitoring.${DOMAIN}
```

When each secret populates, NGINX reloads and the hosts serve valid HTTPS.

---

## 7  Install Rancher (uses pre‑issued cert)

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest && helm repo update
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.${DOMAIN} \
  --set ingress.tls.source=secret \
  --set ingress.tls.secretName=rancher-480-io-tls
```

Login at [**https://rancher.480.io**](https://rancher.480.io) once the deployment rolls out.


---


