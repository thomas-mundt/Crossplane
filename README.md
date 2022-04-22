# Crossplane

## Setup


Install Crossplane
```
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```



Install the Crossplane CLI
```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/release-1.0/install.sh | sh
```


Install the provider
```
# Azure
kubectl crossplane install provider crossplane/provider-azure:v0.14.0

# AWS
# https://github.com/crossplane/provider-aws
kubectl crossplane install provider crossplane/provider-aws:v0.26.1

# Kubernetes
# https://github.com/crossplane-contrib/provider-kubernetes
kubectl crossplane install provider crossplane/provider-kubernetes:main

```

