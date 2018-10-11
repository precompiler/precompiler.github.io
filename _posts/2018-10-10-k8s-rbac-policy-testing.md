---
layout: post
title:  "How to quickly test Kubernetes authorization policy"
date:   2018-10-10 19:30:00
categories: Kubernetes
tags: Kubernetes RBAC impersonate testing
---

* content
{:toc}


> We've created K8S roles, role bindings, how do we test the authentication policies easily and quickly?
> If you use on-prem Kubernetes or PKS, you probably need to use different kubeconfig files to autenticate as different users/groups. If it's EKS, you'll need to assume different roles to test the policies. To get everything setup can take a while.

We can use Kubernetes [user impersonation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation) to quickly test authorization policies.

A quick demo below

##### Apply following demo RBAC config
rbac.yaml
```yaml
- apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: super-user
  rules:
  - apiGroups:
    - '*'
    resources:
    - '*'
    verbs:
    - '*'
  - nonResourceURLs:
    - '*'
    verbs:
    - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-reader-rb
  namespace: default
subjects:
- kind: Group
  name: precompiler:pod-reader
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: super-user-crb
subjects:
- kind: Group
  name: precompiler:super-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: super-user
  apiGroup: rbac.authorization.k8s.io
```
##### Run kubectl with impersonation flags

```bash
# ok
kubectl get pods --as=dummy --as-group="precompiler:pod-reader"
# failed, role bound to default namespace only
kubectl get pods --as=dummy --as-group="precompiler:pod-reader" --namespace=kube-system
# failed, group "precompiler:pod-reader" cannot list services
kubectl get services --as=dummy --as-group="precompiler:pod-reader"
# ok
kubectl get services --as=dummy --as-group="precompiler:super-user"
```
**Note**: The impersonating user must have the ability to perform the “impersonate” verb on the kind of attribute being impersonated (“user”, “group” or "serviceaccount")
sample role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: impersonator
rules:
- apiGroups: [""]
  resources: ["users", "groups", "serviceaccounts"]
  verbs: ["impersonate"]
```
##### kubectl auth can-i
We can also use kubectl auth can-i to check if an action is permitted
```bash
kubectl auth can-i create pods --all-namespaces --as=anyone --as-group="precompiler:super-user"
```
if you get following error, make sure the user impersonated can create selfsubjectaccessreviews
Error from server (Forbidden): selfsubjectaccessreviews.authorization.k8s.io is forbidden: User "system:node:k8s-master2" cannot create selfsubjectaccessreviews.authorization.k8s.io at the cluster scope
