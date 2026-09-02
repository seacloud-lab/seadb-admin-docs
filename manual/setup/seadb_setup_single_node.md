# Setup SeaDB

This page describes how to setup SeaDB in a single node mode. In a single node mode, SeaDB uses embedded Pebble KV storage.

## Run sea-db

Describe how to run SeaDB via Docker image.

The following content is outdated.

----

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
