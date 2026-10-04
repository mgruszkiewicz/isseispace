---
title: "TIL: you don't need glab-cli to connect to Gitlab Agent kubernetes clusters"
date: 2026-10-04T20:02:19+02:00
draft: false
tags: ['en', 'kubernetes']
---
As the title says - you don't actually need the gitlab cli to connect to Kubernetes clusters exposed via Gitlab Agent. It is easier (and creates a short-lived token instead a long-lived one!), but it is not required.

<!--more-->

## How to do it
1. Go to your Gitlab profile preferences
2. Generate a Legacy Personal access token (PAT) with k8s_proxy permission, copy the generated token to somewhere safe
3. Go to the repository which have kubernetes clusters defined (Operate > Kubernetes Clusters)
4. In the table, note the "Agent ID" of the desired cluster.
5. Now in your shell, use kubectl to create a new cluster with a target to gitlab KAS. I put all the things that needs replacing to match your environment in angle brackets.
```
kubectl config set-cluster <DESIRED_CLUSTER_NAME> --server=https://<GITLAB_HOST>/-/kubernetes-agent/k8s-proxy/
```
6. Set credentials to that context - use the token generated in the step 2 and the "Agent ID" from step four.
```
kubectl config set-credentials <DESIRED_CLUSTER_NAME> --token="pat:<AGENT_ID>:<PAT_TOKEN>"
```
7. Create a new kubectl context with the credential and cluster
```
kubectl config set-context <DESIRED_CLUSTER_NAME> --cluster=<DESIRED_CLUSTER_NAME> --user=<DESIRED_CLUSTER_NAME> --namespace=default
```
8. Now you can try to set the new context and validate if you can perform action on the cluster
```
kubectl config use-context <DESIRED_CLUSTER_NAME>
kubectl get pods
```
