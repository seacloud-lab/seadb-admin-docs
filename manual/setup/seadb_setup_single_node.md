# Setup SeaDB

This page describes how to deploy SeaDB in single-node mode with Docker Compose. In this mode, SeaDB uses the embedded Pebble key-value store and exposes its HTTP API on port `8888`.

## Before you start

The commands in this guide require:

- Docker Engine with Docker Compose v2.
- `openssl` to generate a JWT signing secret.
- `wget` to download the Compose file and environment template.
- `curl` to verify the SeaDB API.

The examples use the following paths:

- `/opt/seadb`: Docker Compose files and `.env`.
- `/opt/seadb-data`: persistent SeaDB data mounted into the container at `/shared`.

## 1. Download deployment files

Download `.env` and `seadb.yml` into the same deployment directory:

```bash
mkdir -p /opt/seadb /opt/seadb-data
cd /opt/seadb

wget -O .env https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/env
wget https://seacloud-lab.github.io/seadb-admin-docs/0.9/repo/docker/seadb.yml
chmod 600 .env
```

The template sets `COMPOSE_FILE=seadb.yml`, so Docker Compose automatically uses this file and `.env`. Run all Docker Compose commands below from `/opt/seadb`.

## 2. Configure `.env`

Generate a private random value for SeaDB's JWT signing key:

```bash
openssl rand -hex 32
```

Edit `/opt/seadb/.env`. Set `JWT_PRIVATE_KEY` to the generated value and choose a strong, private password for the initial `root` administrator. Leave the remaining template values at their defaults unless you need to customize the deployment:

```env
JWT_PRIVATE_KEY=<paste-the-generated-value-here>
INIT_SEADB_ROOT_PASSWORD='<set-a-strong-root-password>'
```

The variables used by the Docker deployment are:

| Variable | Description |
| --- | --- |
| `COMPOSE_FILE` | Compose file used for this deployment. |
| `SEADB_IMAGE` | SeaDB Docker image. |
| `SEADB_VOLUME` | Host directory used to persist the contents of `/shared`. |
| `TIME_ZONE` | Container time zone. Defaults to `UTC`. |
| `LOG_TO_STDOUT` | Write SeaDB logs to standard output instead of log files. Defaults to `false`. |
| `JWT_PRIVATE_KEY` | Secret used to sign and verify JWT credentials. Use a private random value of at least 32 characters. |
| `INIT_SEADB_ROOT_PASSWORD` | Password used to create the `root` administrator on first startup. Remove after successful initialization. |

Use single quotes around passwords in `.env` so characters such as `$` and `#` are treated literally.

!!! warning "Keep the JWT signing secret unchanged"
    `JWT_PRIVATE_KEY` is a server signing secret, not a normal REST API key. Keep the same value when recreating or upgrading the container. Changing it invalidates JWTs signed with the old value.

## 3. Start SeaDB

On first startup with a new data directory, the container runs `sea-db init` to create the `root` administrator using `INIT_SEADB_ROOT_PASSWORD`. No separate administrator initialization command is needed. Subsequent starts use the persistent initialization state.

Start SeaDB in detached mode:

```bash
docker compose up -d
```

Check the container state:

```bash
docker compose ps
```

Immediately after startup, the health state may briefly be `starting`. Check the health status again with:

```bash
docker inspect \
  --format='{{.State.Health.Status}}' \
  "$(docker compose ps -q seadb)"
```

The expected status after initialization is:

```text
healthy
```

!!! success "SeaDB container is healthy"
    A `healthy` status means the Docker health check can successfully reach SeaDB's `/ping` endpoint.

## 4. Verify the logs and API

Check the latest SeaDB logs:

With `LOG_TO_STDOUT=false`, SeaDB application logs are stored under `/opt/seadb-data/logs`; the command below shows container startup and initialization output. Set `LOG_TO_STDOUT=true` to include application logs in Docker logs.

```bash
docker compose logs --tail=100 seadb
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
    - You can log in as `root` using the password set during initialization.

At this point, clients can connect to SeaDB at `http://<server-address>:8888`.

## 5. Verify root access and remove the initialization password

Use `seadb-cli` to verify root access. See [SeaDB CLI](../usage/cli.md) for client configuration and usage:

```bash
seadb-cli login --server http://127.0.0.1:8888 --username root
```

Enter the password configured as `INIT_SEADB_ROOT_PASSWORD` when prompted. See [User Management](../usage/user_management.md) to create application users.

After successful initialization and login, back up the root password securely and remove `INIT_SEADB_ROOT_PASSWORD` from `.env`. Recreate the container to remove the initialization password from its environment:

```bash
docker compose up -d
```

The root account and its password remain in persistent storage. Keep `JWT_PRIVATE_KEY` in `.env` unchanged. Changing or restoring `INIT_SEADB_ROOT_PASSWORD` does not reset the password of an already initialized root account.

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
docker compose logs --tail=100 seadb
```

Then test the API manually:

```bash
curl -v http://127.0.0.1:8888/ping
```

Also confirm that port `8888` is not already being used by another service.

### Initialization requires a root password

For a new data directory, set a non-empty `INIT_SEADB_ROOT_PASSWORD` in `.env` before starting SeaDB. If startup reports that this variable is required, set it and recreate the container with `docker compose up -d`.

Do not remove the initialization password until startup and root login have succeeded. Do not delete the persistent data directory to reset a password.

### Existing JWTs stop working after the container is recreated

Confirm that `JWT_PRIVATE_KEY` in `.env` has not changed. Changing it invalidates credentials signed with the previous key.
