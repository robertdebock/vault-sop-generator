# RAFT autopilot snapshots

[source: Automated Integrated Storage Snapshots](https://developer.hashicorp.com/vault/docs/enterprise/automated-integrated-storage-snapshots)
[source: Vault - Standard Procedures](https://developer.hashicorp.com/vault/tutorials/standard-procedures/sop-backup)
[source: Vault - RAFT storage AWS](https://developer.hashicorp.com/vault/tutorials/raft/raft-storage-aws)

## Introduction

Vault supports automated and manual snapshots of the integrated storage (raft) data. These snapshots are taken periodically or as needed and stored in for example an S3 bucket. The snapshots can be used to restore the storage data in case of a disaster.

## Manual snapshots

In order to manually create or restore a snapshot it is required to login to the Vault server.

### Creating a manual snapshot

Create the snapshot with:

```console
vault operator raft snapshot save company-vault.snapshot
```

Upload the snapshot to a remote location, in this case AWS S3:

```console
aws s3 cp company-vault.snapshot s3://vault-snapshots-bucket/company-vault.snapshot
```

### Restoring a manual snapshot

Copy the desired snapshot from the remote location to the Vault server, in this case AWS S3:

```console
aws s3 cp s3://vault-snapshots-bucket/company-vault.snapshot company-vault.snapshot
```

Make sure you are logged into vault.
Restore the snapshot with:

> During the restore process the Vault API will be unavailable!

```console
vault operator raft snapshot restore company-vault.snapshot
```

## Automated snapshots

[source: Vault - Automated backup procedures](https://developer.hashicorp.com/vault/tutorials/standard-procedures/sop-backup#automated-backup-procedures)

Instead of taking a snapshot manually, you can schedule snapshots to be taken automatically at your desired interval. You can create multiple automatic snapshot configurations (daily, weekly, monthly, yearly) to achieve the desired snapshot retention configuration. Automated backups can be configured AFTER the cluster has been initialized. Automated snapshots can be configured manually or with tools like Terraform (i.e. from a seperate Vault configuration repository).

### Automated snapshots and replication

#### DR replication

With Disaster Recovery replication enabled, a snapshot should be taken from *just* the DR *primary* cluster.
This also means that during  a failover, the automated snapshots will have to be reconfigured on the NEW PR primary cluster.

#### PR replication

With Performance Replication enabled, a snapshot should be taken from *both* the PR *primary* and PR *secondary* cluster.

### Configuring automated snapshots manually

Login to the cluster and configure the automated RAFT autopilot snapshot configuration with:

```console
vault write sys/storage/raft/snapshot-auto/config/[:name] [options]
```

### Configuring automated snapshots with Terraform

Run the following Terraform code against the cluster after it has been initialized:

```hcl
data "aws_region" "current" {}

resource "vault_raft_snapshot_agent_config" "hourly" {
  name              = "hourly"
  interval_seconds  = 3600 # 1h
  retain            = 24
  path_prefix       = "/hourly"
  storage_type      = "aws-s3"
  aws_s3_bucket     = "vault-snapshots"
  aws_s3_region     = data.aws_region.current.name
  aws_s3_enable_kms = true
}
```
