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



## Create AWS resources

Link  
- https://doc.crds.dev/github.com/crossplane/provider-aws
- https://github.com/crossplane/provider-aws/tree/master/examples


## VPC


```
apiVersion: ec2.aws.crossplane.io/v1beta1
kind: VPC
metadata:
  name: crossplane-test-vpc
spec:
  forProvider:
    region: eu-central-1
    cidrBlock: 10.10.0.0/16
    enableDnsSupport: true
    enableDnsHostNames: true
    instanceTenancy: default
  providerConfigRef:
    name: default
```



## Subnets


```
---
apiVersion: ec2.aws.crossplane.io/v1beta1
kind: Subnet
metadata:
  name: crossplane-test-subnet-1
spec:
  forProvider:
    region: eu-central-1
    availabilityZone: eu-central-1a
    cidrBlock: 10.10.1.0/24
    vpcIdRef:
      name: crossplane-test-vpc
    tags:
      - key: Environment
        value: Demo
      - key: Name
        value: crossplane-test-subnet-1
    mapPublicIPOnLaunch: true
  providerConfigRef:
    name: default

---
apiVersion: ec2.aws.crossplane.io/v1beta1
kind: Subnet
metadata:
  name: crossplane-test-subnet-2
spec:
  forProvider:
    region: eu-central-1
    availabilityZone: eu-central-1b
    cidrBlock: 10.10.2.0/24
    vpcIdRef:
      name: crossplane-test-vpc
    tags:
      - key: Environment
        value: Demo
      - key: Name
        value: crossplane-test-subnet-2
    mapPublicIPOnLaunch: true
  providerConfigRef:
    name: default
```


## Security Group

```
---
apiVersion: ec2.aws.crossplane.io/v1beta1
kind: SecurityGroup
metadata:
  name: sample-cluster-sg
spec:
  forProvider:
    region: eu-central-1
    vpcIdRef:
      name: crossplane-test-vpc
    vpcId: crossplane-test-vpc
    groupName: my-cool-ekscluster-sg
    description: Cluster communication with worker nodes
    tags:
      - key: Environment
        value: Demo
      - key: Name
        value: sample-cluster-sg
    ingress:
      - fromPort: 80
        toPort: 80
        ipProtocol: tcp
        ipRanges:
          - cidrIp: 10.10.0.0/8
  providerConfigRef:
    name: default

```



## EC2

```
---
apiVersion: ec2.aws.crossplane.io/v1alpha1
kind: Instance
metadata:
  name: sample-instance
spec:
  forProvider:
    region: eu-central-1
    imageId: ami-0006ba1ba3732dd33
    blockDeviceMappings:
      - deviceName: /dev/sdx
        ebs:
          volumeSize: 100
    subnetIdRef:
      name: crossplane-test-subnet-1
  providerConfigRef:
    name: default
```


