
### Creating Vault Injector

Injector allows pods to automatically get secrets from the vault.

```
kubectl apply -f ./injector/injection/
```

Injector would require the api-resource mutatingwebhookconfigurations to be 
enabled. By default this feature is enabled in kubernetes version 17.x+.


##### Kubernetes Auth Policy

For the injector to be authorised to access vault, 
we need to enable K8s auth in the vault

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
  - Create a basic secret in vault manually
  - Inject the secret into the web app pod automatically
- [example-apps/basic-secret](https://github.com/pramode-pandit/hashicorp-vault/tree/main/example-apps/basic-secret)



Dynamic Secrets: Postgres
- Objective:
  - We have a Postgres Database
  - Let's delegate Vault to manage life cycles of our database credentials
  - Deploy an app, that automatically gets it's credentials from vault
- [example-apps/dynamic-postgress](https://github.com/pramode-pandit/hashicorp-vault/tree/main/example-apps/dynamic-postgresql) 

