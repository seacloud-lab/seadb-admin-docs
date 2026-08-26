# FoundationDB single node

FoundationDB offers both installation packages and Docker images. The installation packages start an `fdbmonitor` process (via systemd/launchd) that manages the `fdbserver` and `backup_agent` processes; the Docker image starts the `fdbserver` process directly.

This page describes a single-node FoundationDB deployment using Docker.

## Deploy with Docker Compose

Create a `compose.yml` with the following content:

```yaml
services:
  fdb:
    image: foundationdb/foundationdb:7.3.63
    environment:
      - FDB_NETWORKING_MODE=container
      - FDB_COORDINATOR=fdb
    ports:
      - 4500:4500
    volumes:
      - type: bind
        source: ./log
        target: /var/fdb/logs
      - type: bind
        source: ./data
        target: /var/fdb/data
```

Start the service and initialize the database:

```shell
docker compose up -d
docker compose exec fdb fdbcli --exec 'configure new single ssd'
```

SeaDB needs to read the `fdb.cluster` file. You can copy it from the container, or create it yourself with the correct IP address:

```text
docker:docker@127.0.0.1:4500
```

## Check the status

Connect to the database with `fdbcli` and run `status`, then verify the following items:

- **Storage engine**: configured as `ssd-2`
- **Replication health**: shows `Healthy` after a short while
- **Storage server**: shows sufficient remaining space, not just the default 1 GB

```shell
fdbcli
fdb> status
```

## Troubleshooting

### Storage server shows only 1 GB of remaining space

This usually means the `memory` storage engine is in use. Run the following in `fdbcli` to migrate to the `ssd-2` storage engine:

```text
fdb> configure storage_migration_type=aggressive
fdb> configure ssd
```
