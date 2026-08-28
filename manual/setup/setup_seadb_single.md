# Set up SeaDB on a single node

SeaDB depends on [FoundationDB](https://apple.github.io/foundationdb/).

FoundationDB is a distributed database designed to handle large volumes of structured data across clusters of commodity servers.

## Deploy FoundationDB

For non-production environments, FoundationDB (v7.3.63) can be deployed as a
single-node instance using Docker. For deployments that use native packages,
refer to the [official documentation](https://apple.github.io/foundationdb/getting-started-linux.html).

### Download and modify `.env`

```shell
mkdir -p /opt/seadb
cd /opt/seadb

wget -O .env https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/env
wget https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/fdb.yml
wget https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/fdb.cluster

vim .env
```

Modify `COMPOSE_FILE='seadb.yml'` to `COMPOSE_FILE='fdb.yml'` in the `.env` file.

```shell
COMPOSE_FILE='fdb.yml'
```

### Initialize FoundationDB

```shell
# Start the FoundationDB service
docker compose up -d

# Enter the fdb container
docker exec -it fdb bash

fdbcli
# Configure the ssd-2 storage engine
fdb> configure new single ssd

# Check the FoundationDB status
fdb> status details
```

Confirm that the output reports the `ssd-2` storage engine, healthy replication,
and sufficient free disk space. The mounted `fdb.cluster` file is shared with
the container and may be updated by FoundationDB; do not replace it with an
unrelated cluster file after initialization.

## Deploy SeaDB

### Download and modify `.env`

```bash
cd /opt/seadb

wget https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/seadb.yml

vim .env
```

Modify `COMPOSE_FILE='fdb.yml'` to `COMPOSE_FILE='fdb.yml,seadb.yml'` in the `.env` file.

```shell
COMPOSE_FILE='fdb.yml,seadb.yml'
```

The following fields merit particular attention:

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_SERVER_ACCESS_TOKEN` | A shared secret mapped to SeaDB's `JWT_PRIVATE_KEY` for signing and verifying JWT credentials. Use at least 32 random characters, for example from `pwgen -s 40 1`, and disclose it only to trusted services that issue JWTs for SeaDB. | (required) |
| `SEADB_VOLUME` | The volume directory for SeaDB data. | `/opt/seadb-data` |
| `FDB_VOLUME` | The volume directory for FoundationDB data. | `/opt/fdb-data` |
| `TIME_ZONE` | The time zone. | `UTC` |

### Start SeaDB

Run the following command:

```bash
docker compose up -d

docker logs -f seadb
```

SeaDB is ready when the following log message appears:

```log
seadb | seadb started
```

Verify the HTTP service:

```shell
curl -fsS http://127.0.0.1:8888/ping
```

The expected response is `{"ret":"pong"}`.
