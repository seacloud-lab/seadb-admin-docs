# Overview

SeaDB depends on [FoundationDB](https://apple.github.io/foundationdb/) for storage. FoundationDB is a distributed database designed to handle large volumes of structured data across clusters of commodity servers, and it can be scaled from a single node to thousands of machines.

## Deployment topologies

There are two ways to run SeaDB itself, depending on your scale and reliability requirements:

| Topology | Components | Best for |
| --- | --- | --- |
| [SeaDB single node](build_from_source.md) | FoundationDB (single node) + SeaDB (built from source) | Non-production, evaluation |
| [SeaDB cluster](seadb_cluster_native.md) | Multiple SeaDB nodes + SeaDB-Proxy + Cluster-Manager + etcd + FoundationDB | Production, horizontally-scaled SeaDB service |

FoundationDB, the underlying storage layer, can be deployed as a single node or as a cluster; see [FoundationDB single node](foundationdb_single.md) and [FoundationDB cluster](foundationdb_cluster.md).

!!! note "Two kinds of cluster"
    "FoundationDB cluster" and "SeaDB cluster" are different concepts. The former clusters the underlying storage layer, while the latter clusters the SeaDB service. They can be used together.

## Prerequisites

- **FoundationDB**: SeaDB is tested against FoundationDB `7.3.63`.
- **Memory**: plan for at least 4 GB of available memory per `fdbserver`
  process; actual requirements depend on workload and process configuration.
- **Go** (source builds only): Go 1.24 or later is required to build `sea-db`, `cluster-manager`, and `sea-db-proxy` from source.

## Where to start

- To try SeaDB quickly, build and run a single node from source — see [Build from source](build_from_source.md).
- To deploy the storage layer, see [FoundationDB single node](foundationdb_single.md) and [FoundationDB cluster](foundationdb_cluster.md).
- To deploy the SeaDB service as a cluster, see [SeaDB cluster](seadb_cluster_native.md).
- For the full list of environment variables, see [Environment variables](../configuration/environment_variables.md).
