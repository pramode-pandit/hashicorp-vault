
### Create Standalone vault with tls certificate

##### Generate the self signed certificates

update the cfssl config json
ca-config.json
ca-csr.json
vault-csr.json

Follow the instaruction in `generate_certs.txt` to generate the certs

##### Creating vault deployment

```
kubectl create ns vault
kubectl apply -f ./tls/standalone/
```

##### Initialising Vault

```
kubectl exec  -it -n vault vault-server-0 -- vault operator init
Unseal Key 1: Lb8NOzflB7P//m8T9ytQ55h7SjsmWKD2g5Duy6xeBAVD
Unseal Key 2: ucrecsbW2PvLu+Npdh5UpsjfZTaNR2VdAfD6YcenFROI
Unseal Key 3: oVfbRnHAbWRlR4LxlsAKdmFsbEDxsBExeDIA/K0ynUEZ
Unseal Key 4: xYzqJi6yHevufVn9ADXZTZAYiO+jv8SQC+teaehJ8vZP
Unseal Key 5: Ujy7hIzs+p2Vx/RHNaibeexCY0w0ldxZi//WKxUPVuUj

Initial Root Token: s.xr2ep1s8s5KlObpfJux9XdVP

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```


##### Check Vault Status

```
kubectl exec  -it -n vault vault-server-0 -- vault status
```

##### Unseal 3 times

```
kubectl -n vault exec -it vault-server-0 -- vault operator unseal
kubectl -n vault get pods
```


##### Access Vault UI Dashboard
```
kubectl port-forward -n vault vault-server-0 8200:8200
```
Access with https://localhost:8200

curl -SL https://vault-internal.vault.svc.cluster.local:8200 --cacert ca.pem


##### View logs to check tls is enabled
```
kubectl -n vault logs vault-server-0
==> Vault server configuration:

             Api Address: https://10.244.0.12:8200
                     Cgo: disabled
         Cluster Address: https://vault-0.vault-internal:8201
              Go Version: go1.15.10
              Listener 1: tcp (addr: "0.0.0.0:8200", cluster address: "[::]:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "enabled")
```

-----------------------

### Creating Vault Injector

Injector allows pods to automatically get secrets from the vault.

```
kubectl apply -f ./injector
```

Injector would require the api-resource mutatingwebhookconfigurations to be enabled. By default this feature is enabled in kubernetes version 17.x+.


##### Kubernetes Auth Policy

For the injector to be authorised to access vault, we need to enable K8s auth

```
kubectl -n vault exec -it vault-0 -- sh
vault login
vault auth enable kubernetes

vault write auth/kubernetes/config \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
exit

kubectl -n vault get pods
```

##### Secret Injection Guides


Basic Secrets: Simple Web App
- Objective:
  - Let's create a basic secret in vault manually
  - Application consumes the secret automatically


Dynamic Secrets: Postgres
- Objective:
  - We have a Postgres Database
  - Let's delegate Vault to manage life cycles of our database credentials
  - Deploy an app, that automatically gets it's credentials from vault

