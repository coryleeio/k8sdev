
Whats this?
merge secrets from a vault cluster into a yaml config map mounted in an in memory volume so applications that have config and secrets colocated can still work without their secrets touching the disk.


# Update the 'vault' service account
kubectl apply --filename vault-k8s-auth-rbac.yaml

vault secrets enable -path=kv kv-v2

vault kv put kv/myapp/config username='appuser' password='suP3rsec(et!' ttl='30s'

# Set VAULT_SA_NAME to the service account you created earlier
export VAULT_SA_NAME=$(kubectl get sa -n vault vault -o jsonpath="{.secrets[*]['name']}")

# Set SA_JWT_TOKEN value to the service account JWT used to access the TokenReview API
export SA_JWT_TOKEN=$(kubectl get secret -n vault $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)

# Set SA_CA_CRT to the PEM encoded CA cert used to talk to Kubernetes API
export SA_CA_CRT=$(kubectl get secret -n vault $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)

export K8S_HOST=$(minikube ip)

# Enable the Kubernetes auth method at the default path ("auth/kubernetes")
vault auth enable kubernetes

# Tell Vault how to communicate with the Kubernetes (Minikube) cluster
vault write auth/kubernetes/config token_reviewer_jwt="$SA_JWT_TOKEN" kubernetes_host="https://$K8S_HOST:8443" kubernetes_ca_cert="$SA_CA_CRT"

# Make a policy for our app to read stuff
vault policy write app-policy app-policy.hcl

# Make a vault role that is assumable by the default/app-sa service account.  Grant it the app-policy
vault write auth/kubernetes/role/app-role bound_service_account_names=app-sa bound_service_account_namespaces=default policies=app-policy ttl=24h

# Run the example it will create the service account which can auth

# It will assign the service account to the consul-template init container which will use it to auth to vault and then render the config into an in memory volume.
kubectl apply -f ../secrets_from_vault/deployment.yaml 
kubectl logs -f POD_NAME
