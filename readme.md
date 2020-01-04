# Traefik v2 + cert-manager

## Install traefik v2

```
kubectl apply -f traefik/
```

## Install whoami

```
kubectl apply -f whoami/
```

## Install cert-manager

```
# Install the CustomResourceDefinition resources separately
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
kubectl create namespace cert-manager


echo 'apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system'| kubectl apply -f -
helm init --service-account tiller --upgrade

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.11.0 \
  jetstack/cert-manager
```

- Verifying the installation

```
kubectl get pods --namespace cert-manager
```

- Create cluster issuer + certificate for `whoami.cert.containous.cloud`and `powpow.cert.containous.cloud`

```
kubectl apply -f cert-manager/
```

- Check that the certificate has been generated

```
kubectl describe certificate -n whoami whoami-cert
kubectl describe certificate -n whoami powpow-cert
```

- Check the certificate issuer with the command:

```
echo | openssl s_client -showcerts -servername whoami.cert.containous.cloud -connect whoami.cert.containous.cloud:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer'
```
