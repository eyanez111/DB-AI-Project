
# Create Name space

```
kubectl create namespace ingress-nginx
```
## Use helm to deply ingress controller

Install Ingress‑NGINX

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update

```
# if reinstalling
```bash
helm uninstall ingress-nginx -n ingress-nginx || true
kubectl delete namespace ingress-nginx --grace-period=0 --force --ignore-not-found
kubectl delete validatingwebhookconfiguration,mutatingwebhookconfiguration \
  -l app.kubernetes.io/name=ingress-nginx --ignore-not-found
kubectl delete clusterrole,clusterrolebinding \
  -l app.kubernetes.io/part-of=ingress-nginx --ignore-not-found
kubectl delete ingressclass nginx --ignore-not-found
```

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.image.tag=v1.12.3 \
  --set controller.service.type=LoadBalancer \
  --set controller.service.loadBalancerIP=91.99.154.111 \
  --set controller.publishService.enabled=true
```

```
kubectl -n ingress-nginx rollout status deploy/ingress-nginx-controller
```
The controller’s `LoadBalancer` service should now show the external IP `91.99.154.111`.


# Notes after installation:

The ingress-nginx controller has been installed.
It may take a few minutes for the load balancer IP to be available.
You can watch the status by running 'kubectl get service --namespace ingress-nginx ingress-nginx-controller --output wide --watch'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
