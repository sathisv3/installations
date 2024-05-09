CVAT installation
=================

### clone the cvat github repo and helm repo update
    git clone https://github.com/cvat-ai/cvat.git
    cd helm-chart
    helm dependency update

### Custom values
`sudo vi values.override.yaml`

```yaml
# traefik ingress controller used 
ingress:
  enabled: true
  hostname: cvat.internal.v3analytics.in
  # our default DNS as internal.v3analytics.in 
  # we can use as "*.internal.v3analytics.in"
  annotations:
    cert-manager.io/issuer: letsencrypt-prod
    # cer-manager used for TLs certificate with letsencrypt
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
  tls: true
  tlsSecretName: cloudflare-token-common
  # secrets are reflected across multiple namesapces by using reflector

analytics:
  enabled: true

traefik:
  enabled: true

nuclio:
  enabled: true

```

### create namespace
    kubectl create ns cvat

### cvat installation by using helm 
    helm install cavt helm-chart/ -n cvat --values values.override.yaml
