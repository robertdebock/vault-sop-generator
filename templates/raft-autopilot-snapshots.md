# RAFT autopilot snapshots

[source](https://developer.hashicorp.com/vault/docs/enterprise/automated-integrated-storage-snapshots)
[source](https://developer.hashicorp.com/vault/tutorials/raft/raft-storage-aws)

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

Instead of taking a snapshot manually, you can schedule snapshots to be taken automatically at your desired interval. You can create multiple automatic snapshot configurations (daily, weekly, monthly, yearly) to achieve the desired snapshot retention configuration. Automated backups can be configured AFTER the cluster has been initialized. Automated snapshots can be configured manually or with tools like Terraform (i.e. from a seperate Vault configuration repository).
