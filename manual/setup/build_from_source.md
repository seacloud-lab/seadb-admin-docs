# Build from source

This page describes how to build and run SeaDB from source.

For a single-node deployment, SeaDB uses the embedded Pebble backend by
default. Pebble stores data on the local filesystem and does not require a
running FoundationDB service or cluster file.

## Install FoundationDB client

The `sea-db` binary includes both storage backends. When building from source,
install the FoundationDB client library required by the Go binding, even when
the runtime backend is Pebble. Pebble does not require a running FoundationDB
server or a cluster file.

The client library is required because the current `sea-db` binary includes the
FoundationDB binding, even when Pebble is used at runtime. The FoundationDB
server package is not required for this single-node setup.

### 1. Download FoundationDB

```shell
curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-clients_7.3.63-1_amd64.deb
```

### 2. Install the FoundationDB client

```shell
sudo dpkg -i ./foundationdb-clients_7.3.63-1_amd64.deb
```

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

The following is a minimal single-node example. SeaDB uses Pebble by default.
Adjust the paths and use a persistent secret of at least 32 random characters
for `JWT_PRIVATE_KEY`.

```shell
mkdir -p ./data ./log

export SEADB_LOG_DIR="$PWD/log"
export SEADB_DATA_DIR="$PWD/data"
export JWT_PRIVATE_KEY="<at-least-32-random-characters>"
```

Pebble stores SeaDB's key-value data in `$SEADB_DATA_DIR/base`. It is supported
for single-node deployments only.

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
