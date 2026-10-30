# Backup and restore

SeaDB supports scheduled backup and restore for the Pebble storage backend. Backup is disabled by default.

For FoundationDB backend, SeaDB is not responsible for backup and restore. The FoundationDB admin should setup proper backup and restore mechanism for the FoundationDB cluster.

## Configure backups

SeaDB can be configured to create backups for the Pebble storage backend every day. The backup files may be stored locally or in S3 compatible storage. SeaDB admin can choose the time for backup.

See how to configure scheduled backups with the [Pebble backup environment variables](../configuration/environment_variables.md#pebble-backup).

## Backup files

Backups are ZIP archives containing the Pebble storage state. They use the following filename format:

```text
seadb_backup_YYYYMMDDTHHMMSSZ.zip
```

For local storage it is saved to `SEADB_PEBBLE_BACKUP_LOCAL_DIR`; for S3 storage it is uploaded below the configured prefix. Retention removes archives older than `SEADB_PEBBLE_BACKUP_KEEP_DAYS` from the selected storage.

## Restore

Restore is available only when `SEADB_STORAGE_BACKEND=pebble`. Stop SeaDB before restoring so the data directory is not modified concurrently with a running server. The restore command does not enforce this shutdown requirement.

Restore one local archive:

```bash
sea-db restore --file /var/backups/seadb/seadb_backup_20261029T020000Z.zip
```

Restore an archive from S3:

```bash
sea-db restore --s3 seadb_backup_20261029T020000Z.zip
```

Exactly one of `--file` and `--s3` must be supplied. The local path must point to a regular file. The S3 value is an archive name relative to `SEADB_PEBBLE_BACKUP_S3_PREFIX`; it must not be an absolute object path. S3 restore also requires the bucket, access key ID, and secret key configuration.
