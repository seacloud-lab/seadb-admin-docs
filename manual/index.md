# Introduction

SeaDB is a scalable, high-performance database built on top of [FoundationDB](https://apple.github.io/foundationdb/).

It exposes a REST API for executing SQL, managing bases, users, and base ownership, and it ships with a dedicated command-line tool, `seadb-cli`. SeaDB can run as a single node or as a cluster of multiple SeaDB nodes coordinated by a Cluster-Manager, with SeaDB-Proxy processes in front of them for client access.

Start with the [Setup overview](setup/overview.md) to choose the deployment topology that fits your environment.
