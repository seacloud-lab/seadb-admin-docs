# Overview

SeaDB depends on [FoundationDB](https://apple.github.io/foundationdb/) for storage. FoundationDB is a distributed database designed to handle large volumes of structured data across clusters of commodity servers, and it can be scaled from a single node to thousands of machines.

## Deployment topologies

There are three ways to deploy SeaDB, depending on your scale and reliability requirements:

| Topology | Components | Best for |
| --- | --- | --- |
| [SeaDB single node (Docker)](setup_seadb_single.md) | FoundationDB (single node, Docker) + SeaDB (Docker) | Non-production, evaluation |
| [SeaDB with a FoundationDB cluster (Docker)](setup_seadb_cluster.md) | FoundationDB cluster (multiple nodes) + SeaDB (Docker) | Production, horizontally-scaled storage |
| [SeaDB cluster (etcd)](seadb_cluster_native.md) | Multiple SeaDB nodes + SeaDB-Proxy + Cluster-Manager + etcd + FoundationDB | Production, horizontally-scaled SeaDB service |

The first two topologies run SeaDB itself as a single container. The third topology clusters the SeaDB service itself, using an [etcd](https://etcd.io/) cluster and a Cluster-Manager to coordinate multiple SeaDB nodes, with SeaDB-Proxy processes in front of them for client access.

!!! note "Two kinds of cluster"
    "FoundationDB cluster" and "SeaDB cluster" are different concepts. The former clusters the underlying storage layer, while the latter clusters the SeaDB service. They can be used together.

## Prerequisites

- **FoundationDB**: SeaDB is tested against FoundationDB `7.3.63`.
- **Memory**: each `fdbserver` process needs at least 4 GB of memory; allocate accordingly when running multiple processes per machine.
- **Go** (source builds only): Go 1.24 or later is required to build `sea-db`, `cluster-manager`, and `sea-db-proxy` from source.

## Where to start

- To try SeaDB quickly, follow [SeaDB single node (Docker)](setup_seadb_single.md).
- To deploy the storage layer itself, see [FoundationDB single node](foundationdb_single.md) and [FoundationDB cluster](foundationdb_cluster.md).
- To deploy the SeaDB service as a cluster, see [SeaDB cluster (etcd)](seadb_cluster_native.md).
- To build the binaries from source instead of using Docker, see [Build from source](build_from_source.md).
- For the full list of environment variables, see [Environment variables](../configuration/environment_variables.md).
