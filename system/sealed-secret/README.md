# Securing Secret management in Kubernetes Cluster with Sealed secret
This approach will be install sealed secret controller in kubernetes cluster, to encrypt a Secret resource with Public key from kubernetes cluster, then converted to sealed secret.\
To use sealed secret, kubernetes cluster will decrypt it with private key to get value of the secret
Reference : [bitnami-sealed-secret](https://hub.docker.com/r/bitnami/sealed-secrets-controller)

## Getting Started
Sealed Secret consists of two components :
- a cluster-side controller / operator
- a command utility named `kubeseal`

The `kubeseal` command utility will be use asymmetric encryption to encrypt secrets that only controller can decrypt it.\
\
The `SealedSecret` is look like this :
```
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mysecret
  namespace: mynamespace
spec:
  encryptedData:
    foo: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEq.....
```
\
After decrypted using controller , will be produce `Secret` equally :
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: mynamespace
data:
  foo: YmFy  # <- base64 encoded "bar"
```

## Using at Kubernetes Cluster

### 1. Installing in Kubernetes Cluster with `helm`
For use `SealedSecret` in kubernetes cluster for encrypt `Secret` value, first you must install the package via `helm`, this is will install required components to implement `SealedSecret`\
\
```
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update
helm install sealed-secrets-controller --namespace kube-system sealed-secrets/sealed-secrets
```

### 2. Installing `kubeseal` command utility
This is based on your OS , [here](https://github.com/bitnami-labs/sealed-secrets#installation-from-source) instruction for install for another OS, we will focus in Linux only\
```
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.23.0/kubeseal-0.23.0-linux-amd64.tar.gz
tar -xvzf kubeseal-0.23.0-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

### 3. Creating `SealedSecret`
There two method for creating `SealedSecret` , **first**, you can connect directly to Cluster for encrypt the `Secret`, **secondly**, its secure way getting public key first from the cluster and use `kubeseal` command utility to generate `SealedSecret` with parsing argument `--cert` to add public key location

### 3.1. Creating `SealedSecret` directly to cluster
```
kubectl create secret generic secret-name --dry-run=client --from-literal=foo=bar -o yaml | \
kubeseal \
  --controller-name=sealed-secrets-controller \
  --controller-namespace=kube-system \
  --format yaml > mysealedsecret.yaml
```
The result from `kubeseal` file mysealedsecret.yaml will be contain `encryptedData` of `Secret`

### 3.2 Creating `SealedSecret` not-directly to cluster (secure way)
Why this is secure, because we not need to connect cluster every use `kubeseal` command, but we first generate public key from kubernetes cluster to use later in `kubeseal` command.\
```
kubeseal \
  --controller-name=sealed-secrets-controller \
  --controller-namespace=kube-system \
  --fetch-cert > mycert.pem
```
Then we can start create `SealedSecret` using `kubeseal` with provided public key before.\
We can parsing existing `Secret` file or use `--dry-run` parameter in `kubectl` command.
```
kubectl create secret generic secret-name --dry-run=client --from-literal=foo=bar -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets-controller \
      --controller-namespace=kube-system \
      --format yaml --cert mycert.pem > mysealedsecret_withcert.yaml
```
It make you can use `kubeseal` command without access to kubernetes cluster, more safe way.