# FoundationDB cluster

A FoundationDB cluster consists of multiple `fdbserver` processes that are independent of each other and exchange data over the network.

Before deploying, keep the following points in mind:

- A `fdbserver` process needs at least 4 GB of memory.
- A `fdbserver` process consumes only one CPU core. You can run multiple processes on a single server to make full use of its resources.
- A `fdbserver` process can be bound to any number of roles. The cluster assigns roles automatically based on its current state; only the `coordinator` role must be assigned manually by the administrator.
- Each `fdbserver` process needs a correct `machine-id` and `datacenter-id`, so that multiple replicas of the same data do not fail together.

## 1. Deploy the master node

### `data/fdb.cluster`

Modify the IP address and port as needed. Other nodes read the master node's address from this file when they first start.

```text
docker:docker@127.0.0.1:4500
```

### `image/Dockerfile`

The custom `ENTRYPOINT` and `CMD` are needed to set parameters such as `public-address`, which the default startup script cannot configure.

- `public-address`: the externally accessible address
- `listen-address`: the listening address
- `datacenter-id`: a hexadecimal value of up to 16 characters
- `machine-id`: a hexadecimal value of up to 16 characters

```dockerfile
FROM foundationdb/foundationdb:7.3.63

ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["fdbserver", "--public-address", "127.0.0.1:4500", "--listen-address", "127.0.0.1:4500", "--logdir", "/var/fdb/logs", "--datadir", "/var/fdb/data", "--datacenter-id", "", "--machine-id", ""]
```

### `compose.yml`

Set `network_mode` to `host` to avoid the latency introduced by `docker-proxy` forwarding.

```yaml
services:
  fdb:
    build: ./image
    network_mode: host
    volumes:
      - type: bind
        source: ./log/fdb
        target: /var/fdb/logs
      - type: bind
        source: ./data/fdb
        target: /var/fdb/data
      - type: bind
        source: ./data/fdb.cluster
        target: /var/fdb/fdb.cluster
```

### Start `fdbserver` and initialize the database

```shell
docker compose up -d --build
docker compose exec fdb fdbcli --exec 'configure new single ssd'
```

## 2. Deploy the remaining nodes

The `fdb.cluster` file on every node must match the master node. Other settings can be defined as needed.

## 3. Deploy the `backup_agent` process

The `backup_agent` process is responsible for performing backups.

`data/fdb.cluster` must match the master node.

`image/Dockerfile`:

```dockerfile
FROM foundationdb/foundationdb:7.3.63

ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["backup_agent", "--logdir", "/var/fdb/logs"]
```

`compose.yml`:

```yaml
services:
  backup_agent:
    build: ./image
    network_mode: host
    volumes:
      - type: bind
        source: ./log/backup
        target: /var/fdb/logs
      - type: bind
        source: ./data/backup
        target: /var/fdb/data
      - type: bind
        source: ./data/fdb.cluster
        target: /var/fdb/fdb.cluster
```

## 4. Configure the cluster

### Check the cluster status

```text
fdb> status
```

### Adjust the replica count and the number of coordinators

| Replication mode | Recommended number of coordinators |
| --- | --- |
| `single` | 1 |
| `double` | 3 |
| `triple` | 5 |

```text
fdb> configure double
fdb> coordinators 10.0.4.1:4500 10.0.4.2:4500 10.0.4.3:4500
```
