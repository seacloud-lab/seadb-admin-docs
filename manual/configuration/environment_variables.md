# Environment variables

SeaDB is configured through environment variables. This page documents the
variables consumed by the `sea-db` process in both single-node and cluster
deployments. For cluster coordination, Cluster-Manager, SeaDB-Proxy, and
FoundationDB connection variables, see [Cluster env](cluster_environment_variables.md).

## SeaDB

### General

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_HOST` | The address on which SeaDB listens. | `0.0.0.0` |
| `SEADB_PORT` | The port on which SeaDB listens. | `8888` |
| `SEADB_LOG_DIR` | Existing writable directory in which SeaDB writes its log files. The `sea-db` binary requires a value even when `LOG_TO_STDOUT=true`. | (required) |
| `SEADB_LOG_LEVEL` | SeaDB log level. An invalid value falls back to `info`. | `info` |
| `LOG_TO_STDOUT` | Write SeaDB logs to standard output instead of log files. | `false` |
| `SEADB_SLOW_QUERY_THRESHOLD` | Request duration above which SeaDB writes a query to the slow log. Slow logs are not written separately when `LOG_TO_STDOUT=true`. Accepts a duration such as `1000ms` or `1s`; a plain number is milliseconds. Invalid values fall back to the default. | `1000ms` |
| `SEADB_TRANSACTION_TIMEOUT` | Maximum duration of a transaction. Accepts a duration such as `300s` or `5m`; a plain number is seconds. Invalid values fall back to the default. | `300s` |
| `SEADB_QUERY_PER_MINUTE_LIMIT` | Global API request limit per minute. Set to `0` to disable the limit. | `50000` |
| `SEADB_REQUEST_TIMEOUT` | Maximum duration of an HTTP request. Accepts a duration such as `30s`; a plain number is seconds. Set to `0` to disable the timeout. Invalid values fall back to the default. | `30s` |
| `SEADB_DEV_MOD` | Currently read at startup but has no runtime effect. Do not rely on it for deployment configuration. | `false` |
| `JWT_PRIVATE_KEY` | Secret used to sign and verify JWT credentials. Use a private random value of at least 32 characters. All SeaDB nodes and trusted JWT-issuing services must use the same value. | (required) |

### Profiling

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_ENABLE_PROFILING` | Enable the profiling endpoints. When enabled, `SEADB_PROFILING_PASSWORD` must be set. | `false` |
| `SEADB_PROFILING_PASSWORD` | Password passed as the `password` URL query parameter to access the profiling endpoints. | (required when profiling is enabled) |
| `SEADB_ENABLE_AUTO_HEAP_PROFILING` | When profiling is enabled, automatically write heap profiles when memory pressure reaches the configured threshold. This requires a Linux `/proc` filesystem. | `false` |
| `SEADB_AUTO_HEAP_PROFILING_PRESSURE` | Memory-pressure percentage at which automatic heap profiling runs. Applies only when automatic heap profiling is enabled. | `50` |

### Storage

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_STORAGE_BACKEND` | KV backend used to store SeaDB data. Choose `pebble` or `fdb`. Pebble is for single-node deployments only; SeaDB cluster nodes must use `fdb`. | `pebble` |
| `SEADB_DATA_DIR` | SeaDB working directory. With Pebble, persistent KV data is stored in `base`; SeaDB also uses `tmp` for temporary files such as external-sort files. | (required) |
| `SEADB_KEY_PREFIX` | Prefix applied to SeaDB keys. When a FoundationDB cluster is shared with other applications, use a SeaDB-specific prefix to avoid key collisions. This prefix is also applied by the Pebble backend. | |
| `SEADB_STORAGE_CLEANUP_TIME` | Daily time, in `HH:MM`, at which SeaDB starts cleaning deleted table and index data. Invalid values fall back to the default. | `00:00` |
| `SEADB_UPDATE_BASE_STATS_AT` | Daily time, in `HH:MM`, at which SeaDB updates base storage and table row-count statistics. Invalid values fall back to the default. | `01:00` |

### Backup

Backups are enabled only when `SEADB_STORAGE_SERVER_URL` is set. The backup
schedule uses the server's local time.

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_STORAGE_SERVER_URL` | URL of the storage server that receives SeaDB backups. Setting this value enables background backup processing. | |
| `SEADB_BACKUP_INTERVAL` | Interval between automatic backups. Accepts a duration such as `24h`; a plain number is seconds. Mutually exclusive with `SEADB_BACKUP_AT`. | |
| `SEADB_BACKUP_AT` | Daily automatic-backup time in `HH:MM`. Mutually exclusive with `SEADB_BACKUP_INTERVAL`. When neither schedule variable is set, SeaDB uses this default. Invalid values fall back to the default. | `02:00` |
| `SEADB_KEEP_DAYS` | Number of days to retain backups. When positive, retention is time-based and `SEADB_KEEP_BACKUP_NUM` is ignored. | |
| `SEADB_KEEP_FREQUENCY_DAYS` | With time-based retention, retain every backup from this many recent days; for older backups that remain within `SEADB_KEEP_DAYS`, retain the newest backup for each month. It must not exceed `SEADB_KEEP_DAYS`. | |
| `SEADB_KEEP_BACKUP_NUM` | Number of newest backups to retain when `SEADB_KEEP_DAYS` is not positive. | `3` |

### Metrics

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_METRICS_ENABLE_BASIC_AUTH` | When `true`, protect the metrics endpoint with HTTP basic authentication. When `false`, metrics require a JWT for a SeaDB administrator. | `false` |
| `SEADB_METRICS_USERNAME` | Username accepted by metrics HTTP basic authentication. Set it together with `SEADB_METRICS_PASSWORD` when basic authentication is enabled. | |
| `SEADB_METRICS_PASSWORD` | Password accepted by metrics HTTP basic authentication. Set it together with `SEADB_METRICS_USERNAME` when basic authentication is enabled. | |
| `SEADB_METRICS_INTERVAL` | Interval for refreshing SeaDB's custom metrics. Accepts a duration such as `5s`; a plain number is seconds. Invalid values fall back to the default. | `5s` |

### SQL

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_DEFAULT_RESULT_ROWS` | Default number of rows returned when a query has no `LIMIT` clause. | `100` |
| `SEADB_RESULT_ROWS_HARD_LIMIT` | Maximum number of rows a query can return. | `10000` |
| `SEADB_EXEC_COST_HARD_LIMIT` | Maximum estimated query execution cost. Set to `0` to disable the limit. | `5000000` |
| `SEADB_HASH_JOIN_MEM_LIMIT` | Maximum memory, in bytes, used by a hash join. Set to `0` to disable the limit. | `1073741824` (1 GiB) |
