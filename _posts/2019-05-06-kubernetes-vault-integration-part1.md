---
layout: post
title:  "Kubernetes Vault Integration - Part 1"
date:   2019-05-06 17:00:00
categories: Kubernetes
tags: Kubernetes Vault
---

* content
{:toc}

> There're many ways to get secrets for applications deployed to Kubernetes. For example, you can use Kubernetes secret, but make sure to encrypt your secrets when you deploy them in cloud. Another solution is to use secret stores like Hashicorp Vault. 

I'm gonna show you an end to end Kubernetes Vault integration demo using minikube. This is part 1, which shows you how to quickly setup a Vault server for Kubernetes Vault integration demo. With following scripts, we're going to run a Vault server, enable key value secret engine at a path, and add some secrets to it.

```bash
# create namespace for Vault
kubectl create namespace vault

# deploy Vault
kubectl run vault --image=vault --env="VAULT_DEV_ROOT_TOKEN_ID=vault-root-token" -n vault --port=8200

# expose it
kubectl expose deployment vault --port=80 --target-port=8200 --type=NodePort -n vault

# enable secret path
curl -X POST \
  http://192.168.99.100:30118/v1/sys/mounts/area51 \
  -H 'content-type: application/json' \
  -H 'x-vault-token: vault-root-token' \
  -d '{
	"type": "kv"
}'

# add secrets
curl -X POST \
  http://192.168.99.100:30118/v1/area51/demo-app \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'x-vault-token: vault-root-token' \
  -d '{
		"username": "admin",
		"password": "123456"
}'
```

Note: I'm exposing Vault using NodePort, use kubectl get service ... to find out the static port. You can find the cluster IP in the kube config file minikube generated @ $HOME/.kube/config
