# Build from source

This page describes how to build and run SeaDB from source.

For a single-node deployment, use the Pebble backend by setting
`SEADB_STORAGE_BACKEND=pebble`. To use FoundationDB instead, set
`SEADB_STORAGE_BACKEND=fdb` and configure `SEADB_FDB_CLUSTER_FILE`.

The default backend is `fdb`.

## Choose a storage backend

| Backend | Use when | Runtime storage service |
| --- | --- | --- |
| `pebble` | Running one SeaDB node with local embedded storage | None |
| `fdb` | Running one SeaDB node backed by FoundationDB | FoundationDB |

## Install FoundationDB client

The `sea-db` binary includes both storage backends. When building from source,
install the FoundationDB client library required by the Go binding, even when
the runtime backend is Pebble. Pebble does not require a running FoundationDB
server or a cluster file.

If using the `fdb` backend, install the FoundationDB server package as well. We
recommend using a fixed FoundationDB version, `7.3.63`.

### 1. Download FoundationDB

```shell
curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-clients_7.3.63-1_amd64.deb
```

### 2. Install the FoundationDB client

```shell
sudo dpkg -i ./foundationdb-clients_7.3.63-1_amd64.deb
```

### 3. Install the FoundationDB server (for the `fdb` backend)

Skip this step when using Pebble.

```shell
curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-server_7.3.63-1_amd64.deb
sudo dpkg -i ./foundationdb-server_7.3.63-1_amd64.deb
sudo systemctl start foundationdb
```

If the instance was initialized with the `memory` storage engine, migrate it to
the `ssd-2` storage engine:

```shell
fdbcli --exec "configure storage_migration_type=aggressive"
fdbcli --exec "configure ssd"
```

Plan for at least 4 GB of available memory for the `fdbserver` process. Actual
requirements depend on workload and process configuration.

## Build sea-db

### 1. Install Go

Refer to [https://golang.google.cn/dl/](https://golang.google.cn/dl/). The Go version must be 1.24 or later.

### 2. Configure the Go environment (optional, for users in China)

```shell
go env -w GOPROXY=https://goproxy.cn,direct
```

### 3. Clone the repository

```shell
git clone https://github.com/seafileltd/SeaDB.git
cd SeaDB
```

### 4. Build

```shell
go build -o ./sea-db ./cmd/sea-db
```

## Run sea-db

### 1. Configure the runtime environment

The following is a minimal single-node example using Pebble. Adjust the paths
and use a persistent secret of at least 32 random characters for
`JWT_PRIVATE_KEY`.

```shell
mkdir -p ./data ./log

export SEADB_LOG_DIR="$PWD/log"
export SEADB_DATA_DIR="$PWD/data"
export SEADB_STORAGE_BACKEND="pebble"
export JWT_PRIVATE_KEY="<at-least-32-random-characters>"
```

Pebble stores SeaDB's key-value data in `$SEADB_DATA_DIR/base`; no running
FoundationDB server or cluster file is required. Pebble is for single-node
deployments only. To use FoundationDB instead, install and initialize it as
described above, then use:

```shell
export SEADB_STORAGE_BACKEND="fdb"
export SEADB_FDB_CLUSTER_FILE="/etc/foundationdb/fdb.cluster"
```

### 2. Create an admin account

```shell
./sea-db add-admin -u seadb-admin -p <password>
```

### 3. Run sea-db

```shell
./sea-db
```

Verify that SeaDB is ready:

```shell
curl http://127.0.0.1:8888/ping
```

The expected response is `{"ret":"pong"}`.

## Build and run cluster-manager

### Build

Follow the same Go setup and repository clone steps as for `sea-db`, then run:

```shell
go build -o ./cluster-manager ./cmd/cluster-manager
```

### Run

```shell
./cluster-manager
```

## Build and run sea-db-proxy

### Build

Follow the same Go setup and repository clone steps as for `sea-db`, then run:

```shell
go build -o ./sea-db-proxy ./cmd/sea-db-proxy
```

### Run

```shell
./sea-db-proxy
```

Manager and Proxy require etcd and component-specific environment variables.
See [SeaDB cluster](seadb_cluster_native.md) for their complete runtime
configuration and startup order.

## Troubleshooting

### `Failed to initialize fdb: ... FoundationDB error code 1510 (Disk i/o operation failed)`

Error 1510 is a FoundationDB disk I/O failure. Check filesystem permissions,
free disk space, mount health, and the FoundationDB logs for the failing path.
It does not by itself identify a CPU architecture mismatch.

### `Failed to initialize metabase: ... FoundationDB error code 1031 (transaction timed out)`

This timeout often means that SeaDB cannot reach a healthy FoundationDB
cluster. Check the service, cluster file, network connectivity, and cluster
status:

```shell
# Start FoundationDB if it is stopped.
sudo systemctl start foundationdb

# Verify the local process and cluster status.
systemctl status foundationdb
fdbcli --exec "status details"
```

### `Illegal instruction`

This usually means that a binary was built for a different CPU architecture or
requires CPU instructions unavailable on the host. Install the FoundationDB
packages and build SeaDB for the host architecture. For example, use the
`aarch64` FoundationDB packages on ARM64 Linux instead of the `amd64` packages.
