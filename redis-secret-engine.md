## POC for Redis Secret Engine in Vault using KubeDB Redis

**Deploy Vault Server using KUbeVault**
```yaml
apiVersion: kubevault.com/v1alpha2
kind: VaultServer
metadata:
  name: vault
  namespace: demo
spec:
  replicas: 3
  version: 1.12.1
  allowedSecretEngines:
    namespaces:
      from: All
    secretEngines:
    - gcp
  backend:
    raft:
      storage:
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi
  unsealer:
    secretShares: 5
    secretThreshold: 3
    mode:
      kubernetesSecret:
        secretName: vault-keys
  terminationPolicy: WipeOut
```

**Deploy Standalone Redis using KubeDB**
```yaml
apiVersion: kubedb.com/v1alpha2
kind: Redis
metadata:
  name: standalone-redis
  namespace: demo
spec:
  version: 6.2.5
  disableAuth: true
  storageType: Durable
  storage:
    storageClassName: "standard"
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  podTemplate:
    spec:
      resources:
        requests:
          cpu: "100m"
          memory: "100Mi"
  terminationPolicy: WipeOut
  ```

  **Set up for using vault CLI**
  ```
  kubectl port-forward -n demo svc/vault 8200
 
  export VAULT_ADDR=http://127.0.0.1:8200
 
  export VAULT_TOKEN=(kubectl vault root-token get vaultserver vault -n demo --value-only)
  ```

  **Enable `database` secret engine**
  ```
  vault secrets enable database
  ```
  Output:
  ```
  Success! Enabled the database secrets engine at: database/
  ```

  **Configure Redis Secret Engine**
  ```
  vault write database/config/my-redis-database \
        plugin_name="redis-database-plugin" \
        host="standalone-redis.demo.svc" \
        port=6379 \
        username="default" \
        password="randomPass" \
        allowed_roles="my-*-role"
  ```
  Output:
  ```
  Success! Data written to: database/config/my-redis-database
  ```
  As we have `disableAuth: true` set in the Redis yaml, we can pass any random password when configuring Redis Secret Engine. Otherwise, we need to pass the generated password from db auth secret.

  **Create Role in the Secret Engine**
  ```
  vault write database/roles/my-dynamic-role \
    db_name="my-redis-database" \
    creation_statements='["+@admin"]' \
    default_ttl="1h" \
    max_ttl="10h"
  ```
  Ouput:
  ```
  Success! Data written to: database/roles/my-dynamic-role
  ```

  **Generate Credential**
  ```
  vault read database/creds/my-dynamic-role
  ```
  Output:
  ```
    Key                Value
    ---                -----
    lease_id           database/creds/my-dynamic-role/VLHZdPIS5EcQ2jtv6ffgz6Ww
    lease_duration     1h
    lease_renewable    true
    password           VzJtpM4-oYcPqmCjfwkb
    username           V_ROOT_MY-DYNAMIC-ROLE_WRGGYZTQYYM53EB6VBOE_1670749058
  ```

**Verify the generated credentials**

```
kubectl exec -it -n demo standalone-redis-0 -- sh
```
Then run this comman inside the pod shell
```
redis-cli acl list
```
Output:
```
1) "user V_ROOT_MY-DYNAMIC-ROLE_WRGGYZTQYYM53EB6VBOE_1670749058 on #42625481594af3146cf595274bdf83e580e1122f4eef07e4c1cd25e3a0647485 &* -@all +@admin"
2) "user default on nopass ~* &* +@all"
```

A new user is generated. To login with the generated user we can use `redis-cli auth <username> <password>` command 
```
redis-cli auth V_ROOT_MY-DYNAMIC-ROLE_WRGGYZTQYYM53EB6VBOE_1670749058 VzJtpM4-oYcPqmCjfwkb
```
Output:
```
OK
```
