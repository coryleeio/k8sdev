vault secret enable --path kv kv-v2 

k apply -f ../secrets_from_vault/deployment.yaml 
