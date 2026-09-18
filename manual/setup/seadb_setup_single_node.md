# Setup SeaDB

This page describes how to deploy SeaDB in single-node mode with Docker Compose. In this mode, SeaDB uses the embedded Pebble key-value store and exposes its HTTP API on port `8888`.

## Before you start

The commands in this guide require:

- Docker Engine with Docker Compose v2.
- `openssl` to generate a JWT signing secret.
- `wget` to download the Compose file.
- `curl` to verify the SeaDB API.

The examples use the following paths:

- `/opt/seadb`: Docker Compose files and `.env`.
- `/opt/seadb-data`: persistent SeaDB data mounted into the container at `/shared`.

## 1. Prepare the deployment directory

Create a directory for the Compose files and a separate directory for persistent SeaDB data:

```bash
mkdir -p /opt/seadb /opt/seadb-data
cd /opt/seadb
```

## 2. Create the environment configuration

Generate a private random value for SeaDB's JWT signing key:

```bash
openssl rand -hex 32
```

Create `.env` in `/opt/seadb` and save the generated value as `SEADB_SERVER_ACCESS_TOKEN`:

```env
COMPOSE_FILE=seadb.yml
SEADB_IMAGE=seafileltd/seadb:0.9.0-testing
SEADB_VOLUME=/opt/seadb-data
SEADB_SERVER_ACCESS_TOKEN=<paste-the-generated-value-here>
```

The variables used by the Docker deployment are:

| Variable | Description |
| --- | --- |
| `COMPOSE_FILE` | Compose file used for this deployment. |
| `SEADB_IMAGE` | SeaDB Docker image. |
| `SEADB_VOLUME` | Host directory used to persist the contents of `/shared`. |
| `SEADB_SERVER_ACCESS_TOKEN` | Secret passed to the container as `JWT_PRIVATE_KEY` and used to sign and verify JWT credentials. |

!!! warning "Keep the JWT signing secret unchanged"
    `SEADB_SERVER_ACCESS_TOKEN` is mapped to `JWT_PRIVATE_KEY` inside the SeaDB container. It is a server signing secret, not a normal REST API key. Keep the same value when recreating or upgrading the container. Changing it invalidates JWTs signed with the old value.

Restrict access to `.env` because it contains a secret:

```bash
chmod 600 .env
```

## 3. Download the Docker Compose file

Download `seadb.yml` into the same directory:

```bash
wget -O seadb.yml https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb.yml
```

The Compose file publishes SeaDB on port `8888`, mounts `SEADB_VOLUME` at `/shared`, and includes a health check for `GET /ping`.

Before continuing, validate the resolved Compose configuration:

```bash
docker compose --env-file .env -f seadb.yml config
```

## 4. Create the first administrator

Create the first administrator before starting the SeaDB service. The one-time initialization container accesses the same Pebble data directory as SeaDB, so the regular SeaDB container must not be running at the same time.

Read the administrator password without echoing it to the terminal:

```bash
read -s SEADB_PASSWORD
export SEADB_PASSWORD
echo
```

Then create the administrator:

```bash
docker compose --env-file .env -f seadb.yml run \
  --rm \
  --no-deps \
  -T \
  -e SEADB_DATA_DIR=/shared \
  -e SEADB_LOG_DIR=/shared/logs \
  --entrypoint /opt/seadb/sea-db \
  seadb \
  add-admin \
  --user seadb-admin \
  --password "$SEADB_PASSWORD"
```

A successful initialization prints a message similar to:

```text
level=info msg="Create admin user seadb-admin successful"
```

!!! warning "Do not run add-admin while SeaDB is using the Pebble data directory"
    Run this command only for a new data directory and before starting SeaDB. If SeaDB has already been started, stop it first with:

    ```bash
    docker compose --env-file .env -f seadb.yml stop seadb
    ```

## 5. Start SeaDB

Start SeaDB in detached mode:

```bash
docker compose --env-file .env -f seadb.yml up -d
```

Check the container state:

```bash
docker compose --env-file .env -f seadb.yml ps
```

Immediately after startup, the health state may briefly be `starting`. Check the health status again with:

```bash
docker inspect \
  --format='{{.State.Health.Status}}' \
  "$(docker compose --env-file .env -f seadb.yml ps -q seadb)"
```

The expected status after initialization is:

```text
healthy
```

!!! success "SeaDB container is healthy"
    A `healthy` status means the Docker health check can successfully reach SeaDB's `/ping` endpoint.

## 6. Verify the logs and API

Check the latest SeaDB logs:

```bash
docker compose --env-file .env -f seadb.yml logs --tail=100 seadb
```

After a successful startup, the log contains:

```text
seadb started
```

Then verify the HTTP API directly:

```bash
curl -fsS http://127.0.0.1:8888/ping
```

The expected response is:

```json
{"ret":"pong"}
```

!!! success "SeaDB single-node deployment completed"
    The deployment is ready when all of the following are true:

    - The SeaDB container health status is `healthy`.
    - The SeaDB log shows `seadb started`.
    - `http://127.0.0.1:8888/ping` returns `{"ret":"pong"}`.
    - The initial administrator has been created successfully.

At this point, clients can connect to SeaDB at `http://<server-address>:8888`.

## Persistent data

The Compose file mounts the host directory configured by `SEADB_VOLUME` into the container at `/shared`. With the configuration in this guide, persistent data is stored under:

```text
/opt/seadb-data
```

Recreating or removing the container does not remove this bind-mounted directory. Do not delete `/opt/seadb-data` unless you intend to remove the SeaDB data.

## Troubleshooting

### The container remains in `starting` or becomes `unhealthy`

Check the service logs first:

```bash
docker compose --env-file .env -f seadb.yml logs --tail=100 seadb
```

Then test the API manually:

```bash
curl -v http://127.0.0.1:8888/ping
```

Also confirm that port `8888` is not already being used by another service.

### `add-admin` cannot access the data directory

Make sure the regular SeaDB container is stopped before running the one-time administrator initialization command:

```bash
docker compose --env-file .env -f seadb.yml stop seadb
```

Single-node SeaDB and the `add-admin` process must not access the same Pebble data directory concurrently.

### Existing JWTs stop working after the container is recreated

Confirm that `SEADB_SERVER_ACCESS_TOKEN` in `.env` has not changed. This value is used as SeaDB's `JWT_PRIVATE_KEY`; changing it invalidates credentials signed with the previous key.
