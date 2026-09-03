# Setup SeaDB

This page describes how to setup SeaDB in a single node mode. In a single node mode, SeaDB uses embedded Pebble KV storage.

## Run SeaDB from a Docker image

Use the following Docker Compose configuration to initialize and run a
single-node SeaDB instance.

### Prepare the configuration

Create a directory for the Compose files and the persistent SeaDB data:

```shell
mkdir -p /opt/seadb /opt/seadb-data
cd /opt/seadb
```

Generate a private random value for SeaDB's JWT signing key:

```shell
openssl rand -hex 32
```

Save the output in a `.env` file next to `seadb.yml`. Keep this value
unchanged when recreating the container; changing it invalidates existing JWTs.

```env
COMPOSE_FILE=seadb.yml
SEADB_IMAGE=seafileltd/seadb:0.9.0-testing
SEADB_VOLUME=/opt/seadb-data
SEADB_SERVER_ACCESS_TOKEN=<paste-the-generated-value-here>
```

Restrict access to the file because it contains a secret:

```shell
chmod 600 .env
```

Download the Compose file into the same directory:

```shell
wget -O seadb.yml https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb.yml
```

### Initialize and start SeaDB

Create the first administrator with a one-time container. The SeaDB service
must not be running while this command accesses the Pebble data directory. If
you have already started the service, stop it first with
`docker compose stop seadb`.
Run this command only for a new data directory and before starting SeaDB.
Replace `<password>` with a strong admin password. Do not save the actual
password in `seadb.yml`, `.env`, or a shared script.

```shell
docker compose run --rm --no-deps -T \
  -e SEADB_DATA_DIR=/shared \
  -e SEADB_LOG_DIR=/shared/logs \
  --entrypoint /opt/seadb/sea-db \
  seadb add-admin \
  --user seadb-admin \
  --password '<password>'

docker compose up -d
```

The SeaDB service is ready after the health check passes. Check its logs and
verify the API:

```shell
docker compose logs --tail=100 seadb
curl -fsS http://127.0.0.1:8888/ping
```

The expected response is `{"ret":"pong"}`.
