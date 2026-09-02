# Overview

SeaDB supports two deployment mode:

* A single node mode with **Pebble** as embeded key-value store.
* A cluster mode, which require to use **[FoundationDB](https://apple.github.io/foundationdb/)** as external standalone key-value store.

In most use cases, deploying SeaDB in a single node mode with Pebble backend is enough. For those that want a high scalable SeaDB cluster, FoundationDB is needed as a separate key-value storage engine.

