
# Cert-Manager + GoDaddy Wildcard Certificate Setup

This guide shows you how to install cert-manager in your Kubernetes cluster and configure it to issue a wildcard certificate for your domain (e.g., `*.480.io`) using GoDaddy DNS.

---

## 🧱 Step 1: Install Helm

Helm is required to install the GoDaddy webhook. To install Helm:
(OPTIONAL)
### On Ubuntu/Debian:

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Verify installation:

```
helm version
```

---

## 📥 Step 2: Install Cert-Manager

```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

Verify installation:

```
kubectl get pods -n cert-manager
```

---

## 🛠️ Step 3: Install GoDaddy Webhook

```
helm repo add godaddy-webhook https://snowdrop.github.io/godaddy-webhook
helm install acme-webhook godaddy-webhook/godaddy-webhook \
  --namespace cert-manager \
  --set groupName=acme.480.io
```

---

## 🌍 Step 4 Create ClusterIssuer
### This tells cert-manager to use Let’s Encrypt and the GoDaddy webhook:


Create a file named `clusterissuer.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http
spec:
  acme:
    email: you@480.io
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-http
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply it:

```
kubectl apply -f clusterissuer.yaml
```
# Then do the cluster issuer for Godaddy

```yml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rancher-ingress
  namespace: cattle-system
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-dns
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - rancher.480.io
      secretName: rancher-480-io-tls
  rules:
    - host: rancher.480.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: rancher
                port:
                  number: 443
```

---

## 🏷️ Step 5: Deploy certs

Create a file `wildcard-cert.yaml`:

```yaml
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
  commonName: rancher.480.io
  dnsNames:
  - rancher.480.io
```

Apply it:

```
kubectl apply -f cert-rancher-480.yaml
```
# Do it for all 3 certs and if want to add more create an A record in godaddy and add the manually using these 3 certs as example

Check it's issued:

```
kubectl describe certificate wildcard-480-io -n cert-manager
kubectl get secret wildcard-480-io-tls -n cert-manager
```

---

## 🌐 Step 6: Use the Certificate in Ingress - On different folder than cert-manager

### Once Ingressed deployed following the steps we have in the  ReadMe.md for the ingress-controller directory

Example ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
  namespace: default
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - '*.480.io'
      secretName: wildcard-480-io-tls
  rules:
    - host: app.480.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

---

Enjoy automated HTTPS with Let's Encrypt for all your subdomains! 🎉
