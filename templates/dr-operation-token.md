# DR Operation tokens

A guide on how to use DR Operation tokens.

[source](https://developer.hashicorp.com/vault/tutorials/enterprise/disaster-recovery#dr-operation-token-strategy)
[source](https://developer.hashicorp.com/vault/docs/concepts/integrated-storage/autopilot#replication)

## What is DR Operation token?

After setting up Disaster Recovery replication the DR secondary cluster will be in a read-only state. This means it's no longer possible to log in through normal means or use the cluster in any way.
However sometimes we still need to perform operations on the DR secondary cluster. For example, performing a fail-over or checking/updating the RAFT configuration. DR operation tokens are used to perform these operations.

## DR Operation tokens versus DR batch Operation tokens

There are two types of DR operation tokens: *DR Operation tokens* and *DR batch Operation tokens*.

The most imporant difference between are listed below.

DR Operation tokens:

- Are single-use tokens. Once used they are revoked.
- Require a threshold of unseal/recovery keys to be generated. This can delay things during an outage.
- Are generated reactively. They are generated when needed.
- Are normal tokens.  
  
batch DR Operation tokens:

- Can be used for multiple operations during its lifetime (TTL).
- Do NOT require a threshold of unseal/recovery keys to be generated. Administrator can generate them at any time on their own.
- Are generated proactively to prepare for an outage.
- Are batch tokens.

## Generating a batch DR Operation token

### Configure Vault to generate batch DR Operation tokens

Create the policy:

```console
vault policy write dr-secondary-promotion - <<EOF
path "sys/replication/dr/secondary/promote" {
  capabilities = [ "update" ]
}

# To update the primary to connect
path "sys/replication/dr/secondary/update-primary" {
    capabilities = [ "update" ]
}

# Only if using integrated storage (raft) as the storage backend
# To read the current autopilot status
path "sys/storage/raft/autopilot/state" {
    capabilities = [ "update" , "read" ]
}
EOF
```

Verify the created policy:

```console
vault policy list
```

Create a role "failover-handler" with the policy "dr-secondary-promotion":

```console
vault write auth/token/roles/failover-handler \
    allowed_policies=dr-secondary-promotion \
    orphan=true \
    renewable=false \
    token_type=batch
```

### Create a batch DR Operation token

Create a token with the role "failover-handler":

```console
vault token create -role=failover-handler -ttl=8h
```

### Use the batch DR Operation token

To perform a failover, see the dedicated standard operation procedure (SOP).
Below an example of how to use the batch DR Operation token to read the current autopilot status:

```console
[root@ip-10-154-165-47 ~]# curl --header "X-Vault-Token: hvb.AAAAAQJWB1-5Zo-ncrgVMPPGchizF5Ky7OEFSiW62ppGfGRXdaCtziMj3RdWx75bbe2Nt6ub7zM7sh7TqljV8vEqtGLxVvnl1nLPLdO6I9aHiAsVdvMeo9ebwEBbzOjWojLt2toKTXGMmSq31XuyHYFLWYYtRI64hZu-enPfeYHwmALBozQuhDMW9w3grqEuCttHKXxQKkk" https://vault-dev-use-2.ops-dev.aws.shell.io:443/v1/sys/storage/raft/autopilot/state | jq
```

example output:

```json
{
  "request_id": "dbab954b-6980-4dd1-e36a-2309d82065d8",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "failure_tolerance": 2,
    "healthy": false,
    "leader": "ip-10-154-161-59.aws-us-east-1.sdu-ops-dev.sdu-rds.com",
    "optimistic_failure_tolerance": 2,
    "servers": {
      "ip-10-154-161-148.aws-us-east-1.sdu-ops-dev.sdu-rds.com": {
        "id": "ip-10-154-161-148.aws-us-east-1.sdu-ops-dev.sdu-rds.com",
        "name": "ip-10-154-161-148.aws-us-east-1.sdu-ops-dev.sdu-rds.com",
        "address": "10.154.161.148:8201",
        "node_status": "alive",
        "last_contact": "1.829462398s",
        "last_term": 5,
        "last_index": 259541,
        "healthy": true,
        "stable_since": "2023-04-12T15:49:54.910961312Z",
        "status": "voter",
        "version": "1.11.1",
        "upgrade_version": "1.11.1",
        "node_type": "voter"
      },
==== omitted =====
```

## Generating a DR Operation token

todo.
