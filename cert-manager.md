Cert-manager Installation
=========================
### helm repo add
### For reference <https://cert-manager.io/docs/installation/helm/>
```sh
 helm repo add jetstack https://charts.jetstack.io --force-update
```
### helm repo update
    helm repo update 

### Install CRDs with kubectl 
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.crds.yaml

### Install cert-manager with helm 
    helm install \
    cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --create-namespace \
    --version v1.14.5 \
    #--set installCRDs=true

### for more reference 
<https://technotim.live/posts/kube-traefik-cert-manager-le/>


Reflector Installation
======================
### For reference
<https://technotim.live/posts/k8s-reflector/>

### helm repo for 
    helm repo add emberstack https://emberstack.github.io/helm-charts

### helm repo update 
    helm repo update

### install reflector by using helm 
    helm upgrade --install reflector emberstack/reflector -n cert-manager


Create certificate
==================

### create the cloud flare tocken secret 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-token-common
  namespace: cert-manager
  annotations: 
type: Opaque
stringData:
  cloudflare-token: 1231bc  # cloudflar tocken need to be attach
```

### create the cluster issuer with the cloud flare tocken secret

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod  # letsencrypt-stage
spec:
  acme:  
    email: admin@v3analytics.in
    server: https://acme-v02.api.letsencrypt.org/directory
           #https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: cluster-issuer-key  # auto generated private key
    solvers:
      - dns01:
          cloudflare:
            apiTokenSecretRef:
              name: cloudflare-token-common  # cloud flare tocken secret name
              key: cloudflare-token   # cloud flare tocken secret key 
        selector:
          dnsZones:
            - "internal.v3analytics.in"  # our DNS

```


### create certificate by using the cloudflare tocken secret and the cluster issuer reference

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: new-test
  namespace: cert-manager
spec:
  secretName: cloudflare-token-common  #the secret already created with cloud flare tocken 
  commonName: "*.internal.v3analytics.in"  # dns 
  dnsNames:
    - "internal.v3analytics.in"
    - "*.internal.v3analytics.in"
  issuerRef:
    name: letsencrypt-prod  # name of the cluster issuer
    kind: ClusterIssuer   # kind
    group: cert-manager.io # group
  secretTemplate:
    annotations:
    # reflector should be installed in the same namespace
    # it will reflect the certificate as secret 
      reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
      reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: "certmanager,minio-dev,test,keycloak,grafana,erpnext,harbor,cvat"  # Control destination namespaces
      reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true" # Auto create reflection for matching namespaces
      reflector.v1.k8s.emberstack.com/reflection-auto-namespaces: "certmanager,minio-dev,test,keycloak,grafana,erpnext,harbor,cvat" # Control auto-reflection namespaces

```