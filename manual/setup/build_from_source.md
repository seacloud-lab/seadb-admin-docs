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
systemctl start foundationdb
fdbcli --exec "configure storage_migration_type=aggressive"
fdbcli --exec "configure ssd"
```

FoundationDB has some memory requirements; we recommend allocating at least 4 GB of available memory.

## Build sea-db

### 1. Install Go

Refer to [https://golang.google.cn/dl/](https://golang.google.cn/dl/). The Go version must be 1.24 or later.

### 2. Configure the Go environment (optional, for users in China)

```shell
go env -w GOPROXY=https://goproxy.cn,direct
```

### 3. Clone the repository

The project is located at [https://github.com/seafileltd/SeaDB](https://github.com/seafileltd/SeaDB).

### 4. Build

Change to the project directory, then run:

```shell
go build ./cmd/sea-db
```

## Run sea-db

### 1. Create an admin account

```shell
./sea-db add-admin -u seadb-admin -p <password>
```

### 2. Run sea-db

```shell
./sea-db
```

## Build and run cluster-manager

### Build

Follow the same Go setup and repository clone steps as for `sea-db`, then run:

```shell
go build ./cmd/cluster-manager
```

### Run

```shell
./cluster-manager
```

## Build and run sea-db-proxy

### Build

Follow the same Go setup and repository clone steps as for `sea-db`, then run:

```shell
go build ./cmd/sea-db-proxy
```

### Run

```shell
./sea-db-proxy
```

## Troubleshooting

### `Failed to initialize fdb: ... FoundationDB error code 1510 (Disk i/o operation failed)`

This error indicates an incompatible system. If the host machine is ARM but the SeaDB container is built for amd64, the CPU instruction set cannot be recognized. To resolve it, set up SeaDB in a dedicated ARM container:

1. Create a new container and pull the ARM version of the Ubuntu image (if the host itself is ARM, just pull it directly):

    ```shell
    docker pull ubuntu:24.04
    ```

2. Enter the container:

    ```shell
    docker exec -it <container_name> bash
    ```

3. Download the ARM FoundationDB packages:

    ```shell
    curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-clients_7.3.63-1_aarch64.deb
    curl -L -O https://github.com/apple/foundationdb/releases/download/7.3.63/foundationdb-server_7.3.63-1_aarch64.deb
    ```

4. Install FoundationDB:

    ```shell
    sudo dpkg -i ./foundationdb-clients_7.3.63-1_aarch64.deb
    sudo dpkg -i ./foundationdb-server_7.3.63-1_aarch64.deb
    ```

5. Build `sea-db` following the [Build sea-db](#build-sea-db) section above.

### `Failed to initialize metabase: ... FoundationDB error code 1031 (transaction timed out)`

This error indicates that the FoundationDB service is not running. Start it manually:

```shell
# Start FoundationDB
systemctl start foundationdb

# Verify it started
ps -ef | grep fdbserver
```

### `Illegal instruction`

This error indicates that the instruction set used by the binary cannot be recognized by the CPU. Handle it the same way as the `Disk i/o operation failed` error above.
