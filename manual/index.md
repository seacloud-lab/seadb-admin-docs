# Introduction

SeaDB is a scalable multi-tenant database that support rich data types.

It exposes a REST API for executing SQL, managing bases, users, and base ownership, and it ships with a dedicated command-line tool, `seadb-cli`. SeaDB can run as a single node or as a cluster of multiple SeaDB nodes coordinated by a Cluster-Manager.

A **base** is SeaDB's unit of data organization, roughly equivalent to a database or namespace. Each base has a UUID, and every SQL statement is executed against a specific base.

Start with the [Setup overview](setup/overview.md) to choose the deployment topology that fits your environment.
