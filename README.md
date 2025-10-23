# Vault-Kubernetes-LS
This repository showcases how to deploy **HashiCorp Vault** using **Helm** and **inject secrets into a Kubernetes pod**.



---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Deploying Vault using Helm](#deploying-vault-using-helm)
- [Initializing and Unsealing Vault](#initializing-and-unsealing-vault)
- [Configuring Kubernetes Auth](#configuring-kubernetes-auth)
- [Creating Secrets](#creating-secrets)
- [Creating Service Account and Role](#creating-service-account-and-role)
- [Updating Policies](#updating-policies)
- [Testing Vault Authentication from Pod](#testing-vault-authentication-from-pod)
- [Notes](#notes)

---

## Overview

We use Helm to deploy Vault on Kubernetes in **standalone mode** with the Vault UI enabled. Kubernetes authentication is enabled so pods can authenticate with Vault using their Service Account tokens.

---

## Prerequisites

- Kubernetes cluster 
- `kubectl` installed and configured
- Helm installed

---

## Deploying Vault using Helm

Install Vault using Helm with standalone mode:

```bash
helm install vault hashicorp/vault \
  --set server.dev.enabled=false \
  --set server.standalone.enabled=true \
  --set server.standalone.config="
ui = true
listener \"tcp\" {
  address = \"[::]:8200\"
  tls_disable = \"true\"
}
storage \"file\" {
  path = \"/vault/data\"
}" \
  --set injector.enabled=true \
  --set server.service.type=ClusterIP \
  --set server.service.port=8200
```
## Initializing and Unsealing Vault

Check the Vault pods and exec into them

```bash
kubectl get pods -l app.kubernetes.io/name=vault
kubectl exec -it vault-0 -- /bin/sh
vault operator init  #store the keys somewhere
vault operator unseal
vault login <your-initial-root-token>
```

## Configuring Kubernetes Auth

Enable Kubernetes authentication and configure it

```bash
vault auth enable kubernetes
vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

## Creating Secrets

```bash
vault secrets enable kv

vault kv put kv/my-app/config username=admin password=supersecret

vault kv get kv/my-app/config
```

## Creating Service Account and Role

```bash
# my-app-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  labels:
    app: my-app
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-app-vault-role
rules:
  - apiGroups: [""]
    resources: ["serviceaccounts/token"]
    verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-vault-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: my-app-vault-role
subjects:
  - kind: ServiceAccount
    name: my-app-sa
```
Apply it using
kubectl apply -f my-app-sa.yaml


## Updating Policies

Exec into vault pod

```bash
vault policy write my-app-policy - <<EOF
path "kv/*" {
  capabilities = ["read", "list"]
}
EOF
```

Create Vault role for the Service Account:

```bash
vault write auth/kubernetes/role/my-app-role \
    bound_service_account_names=my-app-sa \
    bound_service_account_namespaces=default \
    policies=my-app-policy \
    ttl=24h
```

## Testing Vault Authentication from Pod
kubectl apply -f testpod.yaml

Exec into the pod and check for directory name vault et voila!!!



