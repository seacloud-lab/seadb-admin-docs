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
| `SEADB_TRANSACTION_TIMEOUT` | Maximum duration of a transaction. Accepts a duration such as `300s` or `5m`; a plain number is seconds. Invalid values fall back to the default. | `300s` |
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

### Metrics

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_METRICS_ENABLE_BASIC_AUTH` | When `true`, the metrics API uses HTTP basic authentication; when `false`, it uses JWT authentication and requires admin privileges. | `false` |
| `SEADB_METRICS_USERNAME` | The username used when HTTP basic authentication is enabled. | (required when basic authentication is enabled) |
| `SEADB_METRICS_PASSWORD` | The password used when HTTP basic authentication is enabled. | (required when basic authentication is enabled) |
| `SEADB_METRICS_INTERVAL` | The statistics update frequency. | `5s` |

### SQL

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_DEFAULT_RESULT_ROWS` | Default number of rows returned when a query has no `LIMIT` clause. | `100` |
| `SEADB_RESULT_ROWS_HARD_LIMIT` | Maximum number of rows a query can return. | `10000` |
| `SEADB_EXEC_COST_HARD_LIMIT` | Maximum estimated query execution cost. Set to `0` to disable the limit. | `5000000` |
| `SEADB_HASH_JOIN_MEM_LIMIT` | Maximum memory, in bytes, used by a hash join. Set to `0` to disable the limit. | `1073741824` (1 GiB) |
