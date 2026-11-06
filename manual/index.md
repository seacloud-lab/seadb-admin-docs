# Introduction

SeaDB is a scalable multi-tenant database that support rich data types.

It exposes a REST API for executing SQL, managing bases, users, and base ownership, and it ships with a dedicated command-line tool, `seadb-cli`. SeaDB runs as a single service with Pebble as its embedded storage backend.

A **base** is SeaDB's unit of data organization, roughly equivalent to a database or namespace. Each base has a UUID, and every SQL statement is executed against a specific base.

Start with the [Setup overview](setup/overview.md) for an introduction to deploying SeaDB.
