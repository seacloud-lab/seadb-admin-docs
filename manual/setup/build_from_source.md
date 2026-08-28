# Build from source

`sea-db` depends on FoundationDB. We recommend using a fixed FoundationDB version, `7.3.63`.

## Install FoundationDB

### 1. Download FoundationDB

```shell
curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-clients_7.3.63-1_amd64.deb
curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-server_7.3.63-1_amd64.deb
```

### 2. Install FoundationDB

```shell
sudo dpkg -i ./foundationdb-clients_7.3.63-1_amd64.deb
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

The following is a minimal local example. Adjust the paths and use a persistent
secret of at least 32 random characters for `JWT_PRIVATE_KEY`. All SeaDB nodes
and trusted services that issue JWTs for SeaDB must use the same value.

```shell
mkdir -p ./data ./log

export SEADB_LOG_DIR="$PWD/log"
export SEADB_DATA_DIR="$PWD/data"
export SEADB_FDB_CLUSTER_FILE="/etc/foundationdb/fdb.cluster"
export JWT_PRIVATE_KEY="<at-least-32-random-characters>"
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
