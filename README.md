### Vault Overview

Video Tutorials
>https://learn.hashicorp.com/vault

Documentation
>https://www.vaultproject.io/docs

GitHub
>https://github.com/hashicorp/vault

<img src="https://github.com/hashicorp/vault/blob/f22d202cde2018f9455dec755118a9b84586e082/Vault_PrimaryLogo_Black.png" width="160" height="160">

Vault is a tool for securely accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, and more. Vault provides a unified interface to any secret, while providing tight access control and recording a detailed audit log.

The key features of Vault are:
- Secure Secret Storage
- Dynamic Secrets
- Data Encryption
- Leasing and Renewal
- Revocation

Once started, the Vault is in a sealed state. Before any operation can be performed on the Vault it must be unsealed. This is done by providing the unseal keys. When the Vault is initialized it generates an encryption key which is used to protect all the data. That key is protected by a master key. By default, Vault uses a technique known as Shamir's secret sharing algorithm to split the master key into 5 shares, any 3 of which are required to reconstruct the master key.

**High Availability**

Vault is primarily used in production environments to manage secrets. As a result, any downtime of the Vault service can affect downstream clients. Vault is designed to support a highly available deploy to ensure a machine or process failure is minimally disruptive.

When running in HA mode, Vault servers have two additional states they can be in: standby and active. For multiple Vault servers sharing a storage backend, only a single instance will be active at any time while all other instances are hot standbys.

**Seal/Unseal**

When a Vault server is started, it starts in a sealed state. In this state, Vault is configured to know where and how to access the physical storage, but doesn't know how to decrypt any of it.

Unsealing is the process of obtaining the plaintext master key necessary to read the decryption key to decrypt the data, allowing access to the Vault.

Prior to unsealing, almost no operations are possible with Vault. For example authentication, managing the mount tables, etc. are all not possible. The only possible operations are to unseal the Vault and check the status of the seal.

**Unsealing**

The unseal process is done by running vault operator unseal or via the API. This process is stateful: each key can be entered via multiple mechanisms on multiple computers and it will work. This allows each shard of the master key to be on a distinct machine for better security.

Once a Vault node is unsealed, it remains unsealed until one of these things happens:

- It is resealed via the API (see below).
- The server is restarted.
- Vault's storage layer encounters an unrecoverable error.

**Auto Unseal**

Auto Unseal was developed to aid in reducing the operational complexity of keeping the unseal key secure. This feature delegates the responsibility of securing the unseal key from users to a trusted device or service. 

**Running Vault on Kubernetes**

Vault can be deployed into Kubernetes using the official HashiCorp Vault Helm chart. The Helm chart allows users to deploy Vault in various configurations:

- Dev : A single in-memory Vault server for testing Vault
- Standalone (default) : a single Vault server persisting to a volume using the file storage backend
- High-Availability (HA) : a cluster of Vault servers that use an HA storage backend such as Consul (default)
- External : a Vault Agent Injector server that depends on an external Vault server

>https://www.vaultproject.io/docs/platform/k8s/helm/run

The Vault Helm chart is the recommended way to install and configure Vault on Kubernetes. In addition to running Vault itself, the Helm chart is the primary method for installing and configuring Vault to integrate with other services such as Consul for High Availability (HA) deployments.

```
$ helm repo add hashicorp https://helm.releases.hashicorp.com
"hashicorp" has been added to your repositories

$ helm search repo hashicorp/vault
NAME            CHART VERSION   APP VERSION DESCRIPTION
hashicorp/vault 0.10.0          1.7.0       Official HashiCorp Vault Chart
```

```
# List the available releases
$ helm search repo hashicorp/vault -l
NAME            CHART VERSION   APP VERSION DESCRIPTION
hashicorp/vault 0.10.0          1.7.0       Official HashiCorp Vault Chart
hashicorp/vault 0.9.1           1.6.2       Official HashiCorp Vault Chart
hashicorp/vault 0.9.0           1.6.1       Official HashiCorp Vault Chart
hashicorp/vault 0.8.0           1.5.4       Official HashiCorp Vault Chart
hashicorp/vault 0.7.0           1.5.2       Official HashiCorp Vault Chart
```

```
# Install version 0.10.0
kubectl create ns hashicorp-vault
helm install vault hashicorp/vault  --version 0.10.0 --namespace hashicorp-vault
helm get all  vault --namespace hashicorp-vault > vault.txt
```
    
**Agent Sidecar Injector**

Injector allows pods to automatically get secrets from the vault.

The Vault Agent Injector alters pod specifications to include Vault Agent containers that render Vault secrets to a shared memory volume using Vault Agent Templates. 

By rendering secrets to a shared volume, containers within the pod can consume Vault secrets without being Vault aware.

The injector is a Kubernetes Mutation Webhook Controller. The controller intercepts pod events and applies mutations to the pod if annotations exist within the request. This functionality is provided by the vault-k8s project and can be automatically installed and configured using the Vault Helm chart.


**Deploy vault server in standalone mode with yamls**

A single Vault server persisting to a volume using the file storage backend

```
kubectl create ns hashicorp-vault
kubectl apply -f .\standalone\
```

In dev mode, vault is initialised by default, but in standalone and HA vault needs to be initialised and unsealed before it can be used.

```
kubectl -n hashicorp-vault exec -it vault-0 -- vault operator init
kubectl -n hashicorp-vault exec -it vault-0 -- vault status
```

**Unseal the vault 3 times**
```
kubectl -n hashicorp-vault exec -it vault-0 -- vault operator unseal
kubectl -n hashicorp-vault get pods
```

**Access Vault UI Dashboard**
```
kubectl port-forward -n vault vault-server-0 8200:8200
```
Access with http://localhost:8200

**Cleanup your deployment**

```
kubectl delete -f ./standalone/
kubectl delete ns hashicorp-vault
```

