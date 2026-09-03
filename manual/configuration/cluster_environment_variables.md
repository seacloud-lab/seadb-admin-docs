# Cluster environment variables

This page documents variables used by SeaDB cluster coordination, the
Cluster-Manager, and SeaDB-Proxy. It also contains the FoundationDB connection
variable used when `SEADB_STORAGE_BACKEND=fdb`. For all `sea-db` variables
shared by single-node and cluster deployments, see [Environment variables](environment_variables.md).

Every SeaDB node in a cluster also needs the general `sea-db` requirements:
`SEADB_STORAGE_BACKEND=fdb`, `SEADB_DATA_DIR`, `SEADB_LOG_DIR`, and
`JWT_PRIVATE_KEY`.

## SeaDB

### FoundationDB

`SEADB_KEY_PREFIX` is documented in the [Storage section](environment_variables.md#storage).

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_FDB_CLUSTER_FILE` | The location of the FoundationDB cluster file. After installing FoundationDB on Linux, the default location is `/etc/foundationdb/fdb.cluster`. | (required) |

### Cluster coordination

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
| `LOG_TO_STDOUT` | Write Cluster-Manager logs to standard output instead of log files. | `false` |
| `SEADB_ETCD_ENDPOINTS` | The etcd server addresses, comma-separated. | (required) |
| `SEADB_ETCD_PREFIX` | The etcd data prefix. Must match SeaDB and the Proxy. | `/seadb` |

## SeaDB-Proxy

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_PROXY_HOST` | The address the program listens on. | `0.0.0.0` |
| `SEADB_PROXY_PORT` | The port the program listens on. | `8888` |
| `SEADB_LOG_DIR` | The log directory. | (required) |
| `SEADB_PROXY_LOG_LEVEL` | The log level. | `info` |
| `LOG_TO_STDOUT` | Write SeaDB-Proxy logs to standard output instead of log files. | `false` |
| `SEADB_ETCD_ENDPOINTS` | The etcd server addresses, comma-separated. | (required) |
| `SEADB_ETCD_PREFIX` | The etcd data prefix. Must match SeaDB and the Manager. | `/seadb` |
| `SEADB_CLUSTER_MANAGER_URL` | The URL of the Cluster-Manager. | (required) |
