# **Vault Notes**

Vault is secret management tool. Vault authorize clients before giving them access to secret. Vaults also monitors which client accessed which secret. Another primary use of vault is it can generate dynamic secrets which are ephemeral and can be revoked after use. 

The key features of Vault are: 

- Secure Secret Storage
- Dynamic Secrets
- Data Encryption
- Leasing and Renewal
- Revocation

## Vault Architecture

![alt text](https://github.com/Shaad7/Vault-Stufff/blob/master/images/vault-architecture.jpg?raw=true 
"Kubernetes Architecture")


## Vault Seal-Unseal

When vault is deployed it is in sealed state. It is unusable in this state and it needs to be unsealed.

### Shamir Seal:
A set of unseal key is used here. Vault generates 5 unseal keys and at least 3 unseal keys are needed to unseal vault. The numbers can be configured.
### Auto-Unseal:
It is developed to reduce complexity of keeping and managing a nubmer of unseal keys secure. Instead of unseal keys, a trusted device or client(HSM or could KMS) is used to Unseal vault.

## Secret Engine

Secret Engine stores data in Vault. There are different types of secret enginer and they can enabled in different paths in Vault. They works like a function which does a operation on a given data and return a result. Secret Engine can generate credentials for cloud providers such as GCP, AWS, Azure and it can also store key value pair in KV Secret Engine. Database Secret Engines for MySQL, MongoDB, Redis etc. can generate credentials for respective DB.


## Lease, Renew and Revoke

When vault generates dynamic secrets for authentication purpose, it creates a lease which contains information about expiration time and renewability. After the expiration period vault revokes the token. The token can be renewed using lease_id and it also can be revoked as needs.

## Authentication

Authentication in Vault is a way to identify and verify the client. Before a client can interact with Vault, it must authenticate using a auth method. After authentication, a token is generated which may have attached policy.

## Token

Token is used to authenticate client in Vault. All the authentication methoid come down to dynamically created tokens. Tokens contain information about client.

## Storage

There are two types of storage in Vault. Integrated storage and external storage. When using external storage, user needs to maintain the storage cluster along with Vault Cluster. There can be different types of external storage like consul(provided by hashicorp), S3, Azure, FileSystem, MySQL etc. On the other hand, integrated storage is introduced later in the Vault. Intergrated storage is managed along with Vault Cluster and to enable HA in uses raft consensus algorithm. The data is written on the disks of Vault server and distributed among Vault nodes using Raft Algorithm.


