---
title: "Backblaze B2 Operator for Kubernetes"
date: 2025-12-27T14:32:39+01:00
draft: false
tags: ['en', 'kubernetes']
---
As you might guess by some of my older entries, I'm a fan of Backblaze B2 object storage.

Some time ago I created my first Kubernetes operator using Operator-SDK in Golang. Before that, I had some experience with python pykube/kopf, but these libraries are pretty much abandoned, so I decided to take a look at the "first party" solution.

I wanted to create Backblaze B2 buckets and keys in the cluster, but I noticed that at the time, there weren't any operators available, so I created my own. It supports creating buckets and keys with ACL. I know - what a diversity of feature from a storage operator

The operator is open-source and can be found on [github.com/mgruszkiewicz/backblaze-operator](https://github.com/mgruszkiewicz/backblaze-operator)

## How to setup?

Setup should be relatively straightforward, you need:

1. create application key in your b2 account with correct permissions or use master application key
2. helm 

## Creating keys

Creating application keys with specific capabilities/permissions is a bit tricky on Backblaze B2, as it is not possible from the web interface - you need to use the B2 CLI to do that.

First, you need to install the b2 cli, that will depend on the platform you are using.
Refer to the [official Backblaze documentation](https://www.backblaze.com/docs/cloud-storage-command-line-tools) on how to do that

then authenticate to your account
```bash
b2 account authorize
```

and finally, we can create a key for operator
```bash
b2 key create operator-blog writeKeys,deleteKeys,listBuckets,listAllBucketNames,readBuckets,writeBuckets,deleteBuckets
```
The output will contain two values, separated by space - the first one will be the application id, the second application key

## Setup operator using Helm

```bash
helm upgrade --install backblaze-operator backblaze-operator \
  --set credentials.b2ApplicationId="your-application-id" \
  --set credentials.b2ApplicationKey="your-application-key" \
  --set credentials.b2Region="us-west-004" \
  --namespace backblaze-operator --create-namespace \
  --repo https://mgruszkiewicz.github.io/helm-charts
```

## Setup operator in FluxCD repository
If you are using GitOps (and you probably should!) to manage your cluster, you can also easily install the operator using FluxCD HelmReleases. As we will need to pass the Backblaze keys, you will need a way to pass securely the keys to the cluster. In my case I prefer to store the secrets in Vault, and I passed them out to cluster via ExternalSecrets, but the method doesn't matter - you can use variable substitution or create a secret manually and reference it in values.

First, create the secret with your Backblaze credentials:
```bash
kubectl create secret generic backblaze-credentials \
  --from-literal=B2_APPLICATION_ID="your-application-id" \
  --from-literal=B2_APPLICATION_KEY="your-application-key"
  --from-literal=B2_REGION="your-b2-region" \
  --namespace backblaze-operator
```

Then apply the following FluxCD configuration:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: isseispace
  namespace: flux-system
spec:
  interval: 1h
  url: https://mgruszkiewicz.github.io/helm-charts
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: backblaze-operator
  namespace: backblaze-operator
spec:
  interval: 5m
  chart:
    spec:
      chart: backblaze-operator
      sourceRef:
        kind: HelmRepository
        name: isseispace
        namespace: flux-system
  values:
    credentials:
      secret:
        useSecret: true
        name: backblaze-credential
```

# Creating an example bucket
After a successful operator installation, we can try to create a new Backblaze B2 bucket from our Kubernetes cluster using CRD.

```yaml
apiVersion: b2.issei.space/v1alpha2
kind: Bucket
metadata:
  name: my-b2-bucket
spec:
  atProvider:
    acl: private
```

The `metadata.name` will be used as a bucket name.  
Note - to create a bucket with `acl: public`, you first need to add a payment method to your Backblaze account, otherwise the creation process will fail.

Let's check the status:
```yaml
❯ kubectl describe buckets.b2.issei.space my-b2-bucket
Name:         my-b2-bucket
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  b2.issei.space/v1alpha2
Kind:         Bucket
Metadata:
  Creation Timestamp:  2025-12-27T15:55:11Z
  Finalizers:
    bucket.b2.issei.space/finalizer
  Generation:        1
  Resource Version:  762
  UID:               3ed908df-373a-452a-825c-b6215ec9d2a0
Spec:
  At Provider:
    Acl:  private
Status:
  At Provider:
    Acl:       private
  Reconciled:  true
Events:
  Type    Reason         Age    From               Message
  ----    ------         ----   ----               -------
  Normal  BucketCreated  2m42s  bucket-controller  Successfully created bucket my-b2-bucket with ACL private
```

Now let's create a new application key that has only access to our new bucket

```yaml
apiVersion: b2.issei.space/v1alpha2
kind: Key
metadata:
  name: my-b2-key
spec:
  atProvider:
    bucketName: my-b2-bucket
    capabilities:
      - deleteFiles
      - listAllBucketNames
      - listBuckets
      - listFiles
      - readBucketEncryption
      - readBucketReplications
      - readBuckets
      - readFiles
      - shareFiles
      - writeBucketEncryption
      - writeBucketReplications
      - writeFiles
  writeConnectionSecretToRef:
      name: new-key

```

and let's check the status:
```
❯ kubectl describe secret new-key
Name:         new-key
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
AWS_ACCESS_KEY_ID:      25 bytes
AWS_SECRET_ACCESS_KEY:  31 bytes
bucketName:             21 bytes
endpoint:               30 bytes
keyName:                9 bytes
```
Now you can connect your application to new Backblaze B2 object storage bucket!

For compatibility with apps that expect S3 object storage, the secret is created with AWS S3 named keys.
