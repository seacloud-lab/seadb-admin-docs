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
| `SEADB_FDB_CLUSTER_FILE` | Path to the FoundationDB cluster file used by the FDB backend. Set this when `SEADB_STORAGE_BACKEND=fdb`. On Linux FoundationDB installations, the usual location is `/etc/foundationdb/fdb.cluster`. | |

### Cluster coordination

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_CLUSTER_ENABLE` | Enable SeaDB node cluster coordination through etcd. | `false` |
| `SEADB_CLUSTER_NODE_ID` | Numeric identifier of this SeaDB node. It must be nonzero, unique across nodes, and stable for the life of the cluster. | (required when cluster mode is enabled) |
| `SEADB_CLUSTER_ETCD_ENDPOINTS` | Comma-separated etcd endpoint addresses used by SeaDB nodes. | (required when cluster mode is enabled) |
| `SEADB_CLUSTER_ETCD_PREFIX` | etcd key namespace for SeaDB cluster coordination. It must match the Manager and Proxy prefix. | `/seadb` |

## Cluster-Manager

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_CLUSTER_MANAGER_HOST` | Address on which Cluster-Manager listens. | `0.0.0.0` |
| `SEADB_CLUSTER_MANAGER_PORT` | Port on which Cluster-Manager listens. | `8890` |
| `SEADB_LOG_DIR` | Directory in which Cluster-Manager writes its log files. Required even when `LOG_TO_STDOUT=true`. | (required) |
| `SEADB_CLUSTER_MANAGER_LOG_LEVEL` | Cluster-Manager log level. An invalid value falls back to `info`. | `info` |
| `LOG_TO_STDOUT` | Write Cluster-Manager logs to standard output instead of log files. | `false` |
| `SEADB_ETCD_ENDPOINTS` | Comma-separated etcd endpoint addresses used by Cluster-Manager. | (required for cluster operation) |
| `SEADB_ETCD_PREFIX` | etcd key namespace used by Cluster-Manager. It must match the SeaDB node and Proxy prefix. | `/seadb` |

## SeaDB-Proxy

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_PROXY_HOST` | Address on which SeaDB-Proxy listens. | `0.0.0.0` |
| `SEADB_PROXY_PORT` | Port on which SeaDB-Proxy listens. | `8888` |
| `SEADB_LOG_DIR` | Directory in which SeaDB-Proxy writes its log files. Required even when `LOG_TO_STDOUT=true`. | (required) |
| `SEADB_PROXY_LOG_LEVEL` | SeaDB-Proxy log level. An invalid value falls back to `info`. | `info` |
| `LOG_TO_STDOUT` | Write SeaDB-Proxy logs to standard output instead of log files. | `false` |
| `SEADB_ETCD_ENDPOINTS` | Comma-separated etcd endpoint addresses used by SeaDB-Proxy. | (required for cluster operation) |
| `SEADB_ETCD_PREFIX` | etcd key namespace used by SeaDB-Proxy. It must match the SeaDB node and Manager prefix. | `/seadb` |
| `SEADB_CLUSTER_MANAGER_URL` | URL of the Cluster-Manager from which the Proxy obtains node information. | (required) |
