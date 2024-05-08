Prometheus Installation
========================

### Add prometheus helm repo
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

### update the helm repo 
    helm repo update

### create namespace for prometheus
    kubectl create namespace prometheus

### install prometheus using helm
    helm install prometheus prometheus-community/kube-prometheus-stack -n prometheus

### need to expose prometheus with standard nodeport portnumber



Grafana installation
====================

### Add grafana helm repo
    helm repo add grafana https://grafana.github.io/helm-charts

### update the helm repo
    helm repo update

### create namespace for grafana
    kubectl create ns grafana

### Install grafana
    helm install grafana grafana/grafana \
    --namespace grafana \
    --set adminPassword='Admin@123'

### expose the application by using ingress

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: grafana-ingress
 namespace: grafana
 annotations:
   cert-manager.io/issuer: "letsencrypt-prod"
   traefik.ingress.kubernetes.io/router.entrypoints: websecure
   traefik.ingress.kubernetes.io/router.tls: "true"
spec:
 tls:
   - hosts:
       - grafana.internal.v3analytics.in
     secretName: cloudflare-token-common
 rules:
   - host: grafana.internal.v3analytics.in
     http:
       paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: grafana
               port:
                 number: 80                
```
    
