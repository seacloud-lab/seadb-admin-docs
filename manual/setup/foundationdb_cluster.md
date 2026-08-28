# FoundationDB cluster

A FoundationDB cluster consists of multiple `fdbserver` processes that are independent of each other and exchange data over the network.

Before deploying, keep the following points in mind:

- Plan for at least 4 GB of available memory per `fdbserver` process. Actual
  memory requirements depend on workload and process configuration.
- An `fdbserver` process can use up to one full CPU core. Run multiple processes
  on a server when the workload should use additional cores.
- An `fdbserver` process can run multiple roles. FoundationDB assigns most
  roles dynamically; administrators explicitly select the coordinator
  processes with the `coordinators` command.
- Configure locality values to describe real failure domains. Processes on the
  same machine should share a `locality-machineid` and `locality-zoneid`;
  processes in the same datacenter should share a `locality-dcid`.

## 1. Deploy the bootstrap node

### `data/fdb.cluster`

Modify the IP address and port as needed. Use an address that every FoundationDB
host can reach. Other nodes use this bootstrap cluster file when they first
start; FoundationDB does not have a permanent master host.

```text
docker:docker@10.0.4.1:4500
```

### `image/Dockerfile`

The custom `ENTRYPOINT` and `CMD` provide explicit routable addresses and
locality values for this deployment.

- `public-address`: the externally accessible address
- `listen-address`: the listening address
- `locality-dcid`: an identifier for the datacenter
- `locality-machineid`: a unique identifier for the machine
- `locality-zoneid`: an identifier for the replication failure zone

```dockerfile
FROM foundationdb/foundationdb:7.3.63

ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["fdbserver", "--public-address", "10.0.4.1:4500", "--listen-address", "0.0.0.0:4500", "--logdir", "/var/fdb/logs", "--datadir", "/var/fdb/data", "--locality-dcid", "dc-1", "--locality-machineid", "fdb-1", "--locality-zoneid", "fdb-1"]
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

The `fdb.cluster` file on every node must identify the same cluster and use
reachable coordinator addresses. Assign each host unique `locality-machineid`
and `locality-zoneid` values. Set `locality-dcid` according to real datacenter
boundaries: hosts in the same datacenter should use the same value, while
independent datacenters should use different values.

## 3. Deploy the `backup_agent` process

The `backup_agent` process is responsible for performing backups.

`data/fdb.cluster` must identify the same cluster as the database nodes.

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

Use an odd number of coordinators. Three or five coordinators are recommended
for production deployments; place them in independent failure domains when
possible.

| Replication mode | Recommended number of coordinators |
| --- | --- |
| `single` | 1 |
| `double` | 3 |
| `triple` | 5 |

```text
fdb> configure double
fdb> coordinators 10.0.4.1:4500 10.0.4.2:4500 10.0.4.3:4500
```
