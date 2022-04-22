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
kubectl get provider.pkg

# AWS
# https://github.com/crossplane/provider-aws
kubectl crossplane install provider crossplane/provider-aws:v0.26.1
kubectl get provider.pkg

# Kubernetes
# https://github.com/crossplane-contrib/provider-kubernetes
kubectl crossplane install provider crossplane/provider-kubernetes:main
kubectl get provider.pkg
```


Now wait until the Provider becomes healthy
```
kubectl get provider.pkg --watch
```

Login to your AWS account


Create creds.conf file
```
AWS_PROFILE=default
echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf

[default]
aws_access_key_id = xxxxx
aws_secret_access_key = xxxxx
```

Create a Provider Secret
```
kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=./creds.conf

Tip: decode secret
echo "<SECRET>" | base64 -d
# mac
openssl base64 -d
# Online
https://www.base64decode.org/
```


Configure the Provider
```
vi ProviderConfig.yaml

apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: creds
```
k apply -f ProviderConfig.yaml





