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
- [Testing Vault Authentication from Pod](#testing-vault-authentication-from-pod)
- [Updating Policies](#updating-policies)
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
## TBC

