# Overview

SeaDB supports two storage backends for its key-value data:

- **Pebble** is an embedded key-value store for a single SeaDB node. It does not
  require a running external storage service.
- **[FoundationDB](https://apple.github.io/foundationdb/)** is an external
  distributed storage backend. It is designed to handle large volumes of
  structured data across clusters of commodity servers, and it can be scaled
  from a single node to thousands of machines.

## Deployment topologies

There are two service topologies. The single-node topology supports either
backend, depending on your scale and reliability requirements:

| Topology | Components | Best for |
| --- | --- | --- |
| [SeaDB single node with Pebble](build_from_source.md) | SeaDB with Pebble (built from source) | Local development, evaluation |
| [SeaDB single node with FoundationDB](build_from_source.md) | FoundationDB (single node) + SeaDB (built from source) | Single-node deployments using FoundationDB |
| [SeaDB cluster](seadb_cluster_native.md) | Multiple SeaDB nodes + SeaDB-Proxy + Cluster-Manager + etcd + FoundationDB | Production, horizontally-scaled SeaDB service |

Pebble is a node-local embedded storage backend. It is supported only for
single-node SeaDB deployments and cannot provide shared storage for a SeaDB
cluster. Select it with `SEADB_STORAGE_BACKEND=pebble`. Use FoundationDB when
deploying SeaDB as a cluster. FoundationDB
itself can be deployed as a single node or as a cluster; see [FoundationDB single
node](foundationdb_single.md) and [FoundationDB cluster](foundationdb_cluster.md).

!!! note "Two kinds of cluster"
    "FoundationDB cluster" and "SeaDB cluster" are different concepts. The former clusters the underlying storage layer, while the latter clusters the SeaDB service. They can be used together.

## Prerequisites

- **FoundationDB server**: required only when using the `fdb` backend. SeaDB is
  tested against FoundationDB `7.3.63`.
- **FoundationDB client library**: required to build the current `sea-db`
  binary from source, including when the runtime backend is `pebble`; see
  [Build from source](build_from_source.md).
- **Memory**: when using FoundationDB, plan for at least 4 GB of available
  memory per `fdbserver` process; actual requirements depend on workload and
  process configuration.
- **Go** (source builds only): Go 1.24 or later is required to build `sea-db`, `cluster-manager`, and `sea-db-proxy` from source.

## Where to start

- To try SeaDB quickly, build and run a single node with Pebble — see [Build from source](build_from_source.md).
- To deploy FoundationDB as the storage layer, see [FoundationDB single node](foundationdb_single.md) and [FoundationDB cluster](foundationdb_cluster.md).
- To deploy the SeaDB service as a cluster, see [SeaDB cluster](seadb_cluster_native.md).
- For the full list of environment variables, see [Environment variables](../configuration/environment_variables.md).
