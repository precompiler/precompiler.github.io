---
layout: post
title:  "Kubernetes Vault Integration - Part 2"
date:   2019-09-07 15:00:00
categories: Kubernetes
tags: Kubernetes Vault
---

* content
{:toc}

By following instructions in Part 1, we have now a Vault space with sample secrets stored in it. Next, we'll enable read access for a namespace, so that applications deployed to the namespace will be able to read secrets from the Vault space.

```bash
# enable kubernetes auth
export VAULT_ADDR=http://192.168.99.100:30524
vault login vault-root-token
vault auth enable -path mycluster kubernetes


# setup token review k8s service account
kubectl create namespace vault-auth
kubectl create serviceaccount vault-auth -n vault-auth
cat <<EOF | kubectl apply -f -
  apiVersion: rbac.authorization.k8s.io/v1beta1
  kind: ClusterRoleBinding
  metadata:
    name: role-tokenreview-binding
    namespace: vault-auth
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: system:auth-delegator
  subjects:
  - kind: ServiceAccount
    name: vault-auth
    namespace: vault-auth
EOF

# register cluster
K8S_SECRET_NAME=$(kubectl get sa vault-auth -n vault-auth -o=jsonpath='{.secrets[0].name}')
K8S_CA_CERT=$(kubectl get secret $K8S_SECRET_NAME -n vault-auth -o=jsonpath='{.data.ca\.crt}' | base64 --decode)
K8S_SA_JWT=$(kubectl get secret $K8S_SECRET_NAME -n vault-auth -o=jsonpath='{.data.token}' | base64 --decode)
vault write auth/mycluster/config token_reviewer_jwt="$K8S_SA_JWT" kubernetes_host='https://192.168.99.100:8443' kubernetes_ca_cert="$K8S_CA_CERT"


# create vault policy
curl -X PUT \
  http://192.168.99.100:30524/v1/sys/policy/readonly \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'x-vault-token: vault-root-token' \
  -d '{
	"policy": "path \"area51/*\" {capabilities = [\"read\", \"list\"]}"
}'

# setup demo space, applications deployed to this space will be able to read secrets
kubectl create namespace demo


# register k8s namespace
vault write auth/mycluster/role/demoreadonly \
        bound_service_account_names=default \
        bound_service_account_namespaces=demo \
        policies=default,readonly \
        ttl=24h

# get vault token
DEMO_SECRET_NAME=$(kubectl get sa default -n demo -o=jsonpath='{.secrets[0].name}')
DEMO_SA_JWT=$(kubectl get secret $DEMO_SECRET_NAME -n demo -o=jsonpath='{.data.token}' | base64 --decode)
curl -X POST \
  http://192.168.99.100:30524/v1/auth/mycluster/login \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d "{
  \"role\": \"demoreadonly\",
  \"jwt\": \"${DEMO_SA_JWT}\"
}"
```

Note: I'm exposing Vault using NodePort, use kubectl get service ... to find out the static port. You can find the cluster IP in the kube config file minikube generated @ $HOME/.kube/config
