# Environment variables

SeaDB is configured through environment variables. This page lists the variables for each component.

## SeaDB

### General

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_HOST` | The address the program listens on. | `0.0.0.0` |
| `SEADB_PORT` | The port the program listens on. | `8888` |
| `SEADB_LOG_DIR` | The log directory of SeaDB. | (required) |
| `SEADB_LOG_LEVEL` | The log level of SeaDB. | `info` |
| `SEADB_SLOW_QUERY_THRESHOLD` | The slow-query threshold of SeaDB. | `1000ms` |
| `SEADB_QUERY_PER_MINUTE_LIMIT` | The global per-minute API call limit. | `50000` |
| `SEADB_DATA_DIR` | The data directory of SeaDB, used to store temporary files such as external sort files. | (required) |
| `SEADB_REQUEST_TIMEOUT` | The request timeout. | `30s` |
| `JWT_PRIVATE_KEY` | The secret used to sign and verify JWT credentials. Use a private random value of at least 32 characters, and configure the same value on every SeaDB node and trusted JWT-issuing service. | (required) |

### FoundationDB

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_KEY_PREFIX` | The key prefix in FoundationDB storage. If the FoundationDB cluster is shared by multiple applications, set a SeaDB-specific prefix to distinguish it from other applications. | |
| `SEADB_FDB_CLUSTER_FILE` | The location of the FoundationDB cluster file. After installing FoundationDB on Linux, the default location is `/etc/foundationdb/fdb.cluster`. | (required) |

### Storage

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_STORAGE_CLEANUP_TIME` | The time at which SeaDB base cleanup starts. Used to clean up deleted table and index data. | `00:00` |

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
| `SEADB_DEFAULT_RESULT_ROWS` | The default row limit returned when no `LIMIT` clause is provided. | `100` |
| `SEADB_RESULT_ROWS_HARD_LIMIT` | The hard limit on the number of rows returned. | `1000` |
| `SEADB_EXEC_COST_HARD_LIMIT` | The query execution cost limit. | `5000000` |

### Cluster

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_CLUSTER_ENABLE` | Whether to enable cluster mode. | `false` |
| `SEADB_CLUSTER_NODE_ID` | The cluster ID of the current node. IDs must be unique across nodes and stay fixed. | (required in cluster mode) |
| `SEADB_CLUSTER_ETCD_ENDPOINTS` | The etcd server addresses, comma-separated. | (required in cluster mode) |
| `SEADB_CLUSTER_ETCD_PREFIX` | The etcd data prefix. Must match the Manager and Proxy. | `/seadb` |

## Cluster-Manager

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_CLUSTER_MANAGER_HOST` | The address the program listens on. | `0.0.0.0` |
| `SEADB_CLUSTER_MANAGER_PORT` | The port the program listens on. | `8890` |
| `SEADB_LOG_DIR` | The log directory. | (required) |
| `SEADB_CLUSTER_MANAGER_LOG_LEVEL` | The log level. | `info` |
| `SEADB_ETCD_ENDPOINTS` | The etcd server addresses, comma-separated. | (required) |
| `SEADB_ETCD_PREFIX` | The etcd data prefix. Must match SeaDB and the Proxy. | `/seadb` |

## SeaDB-Proxy

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_PROXY_HOST` | The address the program listens on. | `0.0.0.0` |
| `SEADB_PROXY_PORT` | The port the program listens on. | `8888` |
| `SEADB_LOG_DIR` | The log directory. | (required) |
| `SEADB_PROXY_LOG_LEVEL` | The log level. | `info` |
| `SEADB_ETCD_ENDPOINTS` | The etcd server addresses, comma-separated. | (required) |
| `SEADB_ETCD_PREFIX` | The etcd data prefix. Must match SeaDB and the Manager. | `/seadb` |
| `SEADB_CLUSTER_MANAGER_URL` | The URL of the Cluster-Manager. | (required) |
