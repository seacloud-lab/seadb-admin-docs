# Environment variables

SeaDB is configured through environment variables. This page documents the
variables consumed by the `sea-db` process in both single-node and cluster
deployments. For cluster coordination, Cluster-Manager, SeaDB-Proxy, and
FoundationDB connection variables, see [Cluster env](cluster_environment_variables.md).

## SeaDB

### General

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_HOST` | The address the program listens on. | `0.0.0.0` |
| `SEADB_PORT` | The port the program listens on. | `8888` |
| `SEADB_LOG_DIR` | The log directory of SeaDB. | (required) |
| `SEADB_LOG_LEVEL` | The log level of SeaDB. | `info` |
| `LOG_TO_STDOUT` | Write SeaDB logs to standard output instead of log files. | `false` |
| `SEADB_SLOW_QUERY_THRESHOLD` | The slow-query threshold of SeaDB. | `1000ms` |
| `SEADB_QUERY_PER_MINUTE_LIMIT` | The global per-minute API call limit. | `50000` |
| `SEADB_REQUEST_TIMEOUT` | The request timeout. | `30s` |
| `JWT_PRIVATE_KEY` | The secret used to sign and verify JWT credentials. Use a private random value of at least 32 characters, and configure the same value on every SeaDB node and trusted JWT-issuing service. | (required) |

### Storage

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_STORAGE_BACKEND` | The KV backend used to store data. Choose either `fdb` or `pebble`. | `pebble` |
| `SEADB_DATA_DIR` | The data directory of SeaDB, used to store temporary files such as external sort files. | (required) |
| `SEADB_KEY_PREFIX` | The key prefix used by SeaDB's FDB and Pebble storage backends. If the FoundationDB cluster is shared by multiple applications, set a SeaDB-specific prefix to distinguish it from other applications. | |
| `SEADB_STORAGE_CLEANUP_TIME` | The time at which SeaDB base cleanup starts. It is used to clean up deleted table and index data. | `00:00` |
| `SEADB_UPDATE_BASE_STATS_AT` | Daily time, in `HH:MM`, at which SeaDB updates base storage and table row-count statistics. Invalid values fall back to the default. | `01:00` |

### Pebble backup

Pebble backups are disabled by default. When enabled, the backup worker creates scheduled ZIP archives and stores them locally or in S3. See [Backup and restore](../setup/backup_restore.md) for the backup and restore procedure.

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_PEBBLE_BACKUP_ENABLE` | Enable scheduled Pebble backups. | `false` |
| `SEADB_PEBBLE_BACKUP_AT` | Daily backup time in `HH:MM` format. Required when backups are enabled. | (required when enabled) |
| `SEADB_PEBBLE_BACKUP_KEEP_DAYS` | Delete backups older than this number of days. | `3` |
| `SEADB_PEBBLE_BACKUP_STORAGE` | Backup storage. Choose `local` or `s3`. | `local` |
| `SEADB_PEBBLE_BACKUP_LOCAL_DIR` | Destination directory for local backups. | (required for `local`) |
| `SEADB_PEBBLE_BACKUP_S3_PREFIX` | Prefix under which S3 backups are uploaded. | |
| `SEADB_PEBBLE_BACKUP_S3_HOST` | S3-compatible service endpoint. | |
| `SEADB_PEBBLE_BACKUP_S3_AWS_REGION` | AWS or S3-compatible service region. | |
| `SEADB_PEBBLE_BACKUP_S3_USE_HTTPS` | Whether to use HTTPS. | |
| `SEADB_PEBBLE_BACKUP_S3_BUCKET` | S3 bucket for backups. | (required for `s3`) |
| `SEADB_PEBBLE_BACKUP_S3_PATH_STYLE_REQUEST` | Whether to use path-style requests. | |
| `SEADB_PEBBLE_BACKUP_S3_KEY_ID` | S3 access key ID. | (required for `s3`) |
| `SEADB_PEBBLE_BACKUP_S3_KEY` | S3 secret access key. | (required for `s3`) |
| `SEADB_PEBBLE_BACKUP_S3_USE_V4_SIGNATURE` | Whether to use V4 request signatures. | |
| `SEADB_PEBBLE_BACKUP_S3_SSE_C_KEY` | Server-side encryption customer key. | |

`SEADB_PEBBLE_BACKUP_STORAGE` must be `local` or `s3`. S3 storage requires the bucket, access key ID, and secret access key variables shown above. Backups are uploaded below the configured `SEADB_PEBBLE_BACKUP_S3_PREFIX`, and archives older than `SEADB_PEBBLE_BACKUP_KEEP_DAYS` are removed from the selected storage.

### Metrics

SeaDB exposes Prometheus compatible metrics through `/metrics` endpoint. The options here controls the metrics.

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_METRICS_ENABLE_BASIC_AUTH` | When `true`, the metrics API uses HTTP basic authentication; when `false`, it uses JWT authentication and requires admin privileges. | `false` |
| `SEADB_METRICS_USERNAME` | The username used when HTTP basic authentication is enabled. | (required when basic authentication is enabled) |
| `SEADB_METRICS_PASSWORD` | The password used when HTTP basic authentication is enabled. | (required when basic authentication is enabled) |
| `SEADB_METRICS_INTERVAL` | The statistics update frequency. | `5s` |

### SQL

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_DEFAULT_RESULT_ROWS` | The default row limit returned when no `LIMIT` clause is provided. | `100` |
| `SEADB_RESULT_ROWS_HARD_LIMIT` | The hard limit on the number of rows returned. | `10000` |
| `SEADB_EXEC_COST_HARD_LIMIT` | The query execution cost limit. | `5000000` |
