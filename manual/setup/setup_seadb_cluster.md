# Set up a SeaDB cluster

SeaDB depends on [FoundationDB](https://apple.github.io/foundationdb/).

FoundationDB is a distributed database designed to handle large volumes of structured data across clusters of commodity servers.

## Deploy a FoundationDB cluster

This guide uses a three-node FoundationDB (v7.3.63) setup as an example: cluster-fdb1 (192.168.0.10), cluster-fdb2 (192.168.0.11), cluster-fdb3 (192.168.0.12). Each node runs two fdbserver processes, and each fdbserver process uses its own dedicated SSD data disk (`/fdb1` and `/fdb2`). Unless otherwise specified, the following steps should be performed on all three nodes. For more information, please refer to the [official documentation](https://apple.github.io/foundationdb/building-cluster.html).

### Install FoundationDB

Install FoundationDB on each node according to the [official documentation](https://apple.github.io/foundationdb/getting-started-linux.html).

Stop the service, create the target directories, and migrate any existing data.

```shell
# Stop the service
systemctl stop foundationdb || service foundationdb stop

# Create data directories
mkdir -p /fdb1/foundationdb/data/4500 /fdb1/foundationdb/log/4500
mkdir -p /fdb2/foundationdb/data/4501 /fdb2/foundationdb/log/4501

# Migrate existing data
rsync -aHAX --numeric-ids /var/lib/foundationdb/data/4500/ /fdb1/foundationdb/data/4500/
# Skip this step if the second fdbserver process has never been started.
rsync -aHAX --numeric-ids /var/lib/foundationdb/data/4501/ /fdb2/foundationdb/data/4501/

# Update permissions
chown -R foundationdb:foundationdb /fdb1/foundationdb /fdb2/foundationdb
chmod 700 /fdb1/foundationdb/data/4500 /fdb2/foundationdb/data/4501
chmod 755 /fdb1/foundationdb/log/4500 /fdb2/foundationdb/log/4501
```

### Modify `foundationdb.conf`

Specify `datadir` and `logdir` separately for each `fdbserver` process. Assign
an appropriate process `class` to each one.

Keep the default `[fdbserver]` section unchanged. Update or replace the existing
`[fdbserver.4500]` section, then add `[fdbserver.4501]`:

```shell
[fdbserver.4500]
class  = transaction
datadir = /fdb1/foundationdb/data/4500
logdir  = /fdb1/foundationdb/log/4500

[fdbserver.4501]
class  = storage
datadir = /fdb2/foundationdb/data/4501
logdir  = /fdb2/foundationdb/log/4501
```

### Start the FoundationDB service

Confirm that the FoundationDB processes are running, then verify the cluster
status and the class and assigned roles of each process.

```shell
systemctl start foundationdb || service foundationdb start

ss -lntp | egrep ':4500|:4501' || true

fdbcli --exec "status json" \
| jq -r '.cluster.processes
         | to_entries[]
         | "\(.value.address)\tclass=\(.value.class_type // "unset")\troles=\((.value.roles // [])|map(.role)|unique|join(","))"'
```

Sample output:

```text
192.168.0.12:4501   class=storage  roles=ratekeeper,storage
192.168.0.11:4500   class=transaction  roles=cluster_controller,coordinator,log,resolver
192.168.0.12:4500   class=transaction  roles=commit_proxy,coordinator,grv_proxy,log
192.168.0.10:4500   class=transaction  roles=commit_proxy,coordinator,log,master
192.168.0.10:4501   class=storage  roles=data_distributor,storage
192.168.0.11:4501   class=storage  roles=consistency_scan,storage
```

For a more detailed status report, run:

```shell
fdbcli
fdb> status details
```

FoundationDB assigns most roles dynamically, so the exact role distribution
may differ from the sample output. The `master` label is also a dynamically
assigned role; it does not identify a permanent master node.

### Configure FoundationDB for external access

Run the following on `cluster-fdb1`.

```shell
# Update the cluster file to use an externally reachable address.
python3 /usr/lib/foundationdb/make_public.py

cat /etc/foundationdb/fdb.cluster
# DO NOT EDIT!
# This file is auto-generated, it is not to be edited by hand
description:cluster-id@192.168.0.10:4500
```

### Add the other FoundationDB nodes to the cluster

Copy the cluster file from an existing node in the cluster to the corresponding path on the new node (overwrite it), then restart.

On `cluster-fdb2` and `cluster-fdb3`, run:

```shell
scp root@192.168.0.10:/etc/foundationdb/fdb.cluster /etc/foundationdb/fdb.cluster

systemctl restart foundationdb || service foundationdb restart
```

Check the cluster status to confirm that each machine has joined the cluster and that the process count is correct.

```shell
fdbcli --exec "status details"
```

Key output:

```shell
Cluster:
  FoundationDB processes - 6
  Zones                  - 3
  Machines               - 3
```

If a new node fails to join the cluster, it is usually because it was previously started with an old configuration. In that case, stop the service, clean the data directories, and rejoin:

!!! danger
    The following `rm -rf` commands permanently delete the local FoundationDB
    data on the node. Run them only on a node that is being rejoined and only
    after confirming that the remaining cluster has healthy replicas.

```shell
systemctl stop foundationdb || service foundationdb stop

scp root@192.168.0.10:/etc/foundationdb/fdb.cluster /etc/foundationdb/fdb.cluster
chown foundationdb:foundationdb /etc/foundationdb/fdb.cluster
chmod 664 /etc/foundationdb/fdb.cluster
chmod 775 /etc/foundationdb

rm -rf /fdb1/foundationdb/data/4500/*
rm -rf /fdb2/foundationdb/data/4501/*

chown -R foundationdb:foundationdb /fdb1/foundationdb /fdb2/foundationdb

systemctl start foundationdb || service foundationdb start

ss -lntp | egrep '4500|4501'

# Check cluster status.
fdbcli --exec "status details"
```

### Configure coordinators

After all nodes have joined, configure an odd number of coordinators to improve
fault tolerance.

Run the following on any node. Use one process on each of three separate nodes
as a coordinator, such as port 4500 in this example:

```shell
fdbcli
fdb> coordinators 192.168.0.10:4500 192.168.0.11:4500 192.168.0.12:4500
```

### Adjust the redundancy mode and storage engine

Run on any node.

```shell
fdbcli
fdb> configure storage_migration_type=aggressive
fdb> configure double ssd
fdb> status details
```

## Deploy SeaDB

### Download and modify `.env`

Deploy SeaDB on a separate node using Docker.

Download `.env`, `seadb.yml`, and `fdb.cluster` into a deployment directory,
such as `/opt/seadb`:

```bash
mkdir -p /opt/seadb
cd /opt/seadb

wget -O .env https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/env
wget https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/seadb.yml
wget https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb/fdb.cluster

vim .env
```

The following fields merit particular attention:

| Variable | Description | Default |
| --- | --- | --- |
| `SEADB_SERVER_ACCESS_TOKEN` | A shared secret mapped to SeaDB's `JWT_PRIVATE_KEY` for signing and verifying JWT credentials. Use at least 32 random characters, for example from `pwgen -s 40 1`, and disclose it only to trusted services that issue JWTs for SeaDB. | (required) |
| `SEADB_VOLUME` | The volume directory for SeaDB data. | `/opt/seadb-data` |
| `TIME_ZONE` | The time zone. | `UTC` |

Replace the contents of `fdb.cluster` with the contents of
`/etc/foundationdb/fdb.cluster` from any FoundationDB node:

```bash
vim fdb.cluster
```

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
