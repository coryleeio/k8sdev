
Whats this?
WIP: merge secrets from a vault cluster into a yaml config map mounted in an in memory volume so applications that have config and secrets colocated can still work without their secrets touching the disk.



vault secret enable --path kv kv-v2 
k apply -f ../secrets_from_vault/deployment.yaml 


