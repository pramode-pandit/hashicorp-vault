### Basic Secret Injection

This is the manual way. we have to create a role and policy and have to put the secret in the vault. This secret will be access upon by the pod by vault-injector.

lets setup a role in the vault to secure which SA can access the secret.

```
#Create a role for our app

kubectl -n vault exec -it vault-server-0 -- sh 

vault write auth/kubernetes/role/basic-secret-role \
   bound_service_account_names=basic-secret \
   bound_service_account_namespaces=my_apps \
   policies=basic-secret-policy \
   ttl=1h
```

The above maps our Kubernetes service account, used by our pod, to a policy.

Now lets create the policy to map our service account to a bunch of secrets


```
kubectl -n vault exec -it vault-server-0 sh
cat <<EOF > /home/vault/app-policy.hcl
path "secret/basic-secret/*" {
  capabilities = ["read"]
}
EOF
vault policy write basic-secret-policy /home/vault/app-policy.hcl
exit
```

Now our service account for our pod can access all secrets under `secret/basic-secret/*`

Lets create some secrets.


```
kubectl -n vault exec -it vault-0 sh 
vault secrets enable -path=secret/ kv
vault kv put secret/basic-secret/helloworld username=dbuser password=sUp3rS3cUr3P@ssw0rd
exit
```

Lets deploy our app and see if it works:

```
kubectl create ns my_apps
kubectl -n my_apps apply -f ./example-apps/basic-secret/deployment.yaml
kubectl -n my_apps get pods
```
```
NAME                            READY   STATUS    RESTARTS   AGE
basic-secret-6699bfc678-4wtmt   2/2     Running   0          55m
```

