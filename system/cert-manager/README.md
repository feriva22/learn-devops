# How to installing Cert-manager in Kubernetes Clusters

## Getting Started 

### 1. Adding repository jetstack
`helm repo add jetstack https://charts.jetstack.io`

### 2. Update Repository
`helm update`

### 3. Install CRD for pre-requiment installation
For installation cert-manager, some CRD (Custom Resource Definition) needed, so we can install it with manifest execute with `kubectl` or add `--set installCRDs=true` flag when install cert-manager with helm\

`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml`

### 4. Install cert-manager with helm
This is if you want install cert-manager
`helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.12.0 \`

### 5. Validation installation 
To validation if chart cert-manager success installed, you can use\
`helm list -n cert-manager`


## Issuers and Certificates
`cert-manager` mainly uses two different custom Kubernetes resources - known as `CRDs` - to configure and control how it operates, as well as to store state. These resources are Issuers and Certificates.
### Issuers
An Issuer defines how cert-manager will request TLS certificates. Issuers are specific to a single namespace in Kubernetes, but there's also a ClusterIssuer which is meant to be a cluster-wide version. Example of Issuers is like Lets encrypt

### Certificates
Certificates resources allow you to specify the details of the certificate you want to request. They reference an issuer to define how they'll be issued.

## Create new Certificate with Letsencrypt Issuers
We'll set up two issuers for Let's Encrypt in this example: staging and production.\
in production there some hit limit, so we focus on staging first, after that use production\
You can check in file `issuer/letsencrypt-issuer-staging.yaml`\
If you can check in that file, in above we have HTTP-01 Challenge, we point it to ingress class Nginx, if you have another ingress class installed you can change it\
Deploy it with command
`kubectl apply -f issuer/letsencrypt-issuer-staging.yaml`\
This issuer must be placed in namespace where certificate created, if your application deployed in another namespace then change the namespace
If you want place the issuer in cluster-wide without specific namespace, then use `ClusterIssuer` CRD
