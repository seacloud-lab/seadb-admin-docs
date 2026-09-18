# Accessing Base

This page describes how clients authenticate to SeaDB and how to access the
data in a base.

## Authentication

SeaDB authenticates every request using the `Authorization` header. Three
credential types are supported.

### Basic authentication

Send a username and password as HTTP Basic authentication:

```text
Authorization: Basic <base64(username:password)>
```

For username `admin` and password `123456`, the header is:

```text
Authorization: Basic YWRtaW46MTIzNDU2
```

Basic authentication is primarily intended for first-time setup. Prefer API
keys for programmatic access.

### API key

Send a previously created API key:

```text
Authorization: ApiKey <encoded>
```

`<encoded>` is the `key_id:key` pair returned by the server as `encoded` when
the key was created (see [User Management](user_management.md)). The API key
must belong to the user and must not be expired.

The CLI handles this for you: `seadb-cli login` creates an API key and stores
its `encoded` value locally, and subsequent commands attach it automatically.
To authenticate an API client directly, create an API key through the
management API and send it in the header above.

### JWT (base-scoped)

A user can mint a JWT scoped to a single base, which lets a client access that
base without forwarding the user's password or API key. The JWT can grant
table-level and operation-level permissions:

- `read` — read rows and metadata.
- `read-write` — read, insert, update, and delete rows (no metadata changes).
- `all` — any operation, including metadata changes.

A JWT carries a `base_id`, a `username`, the table names it covers (`*` matches
any table), and an expiration time (3 days by default). The user must own the
base (or be an `admin`) to mint a JWT for it. Send the token with:

```text
Authorization: Bearer <jwt_token>
```

Refer to the [API reference](https://seadb-api.readme.io) for the endpoint that
mints base JWTs.

## Access data

### Via the CLI

After logging in, run SQL against a base with `seadb-cli sql`:

```bash
seadb-cli sql --base <base> -e "SELECT title FROM Tasks WHERE done = false;"
```

The CLI supports `SELECT`, `EXPLAIN`, `INSERT`, `REPLACE`, `UPDATE`, `DELETE`,
and DDL statements. See the [SeaDB CLI](cli.md) for details.

### Via the SQL API

Execute a SQL statement against a base by sending it to the query endpoint
(refer to the [API reference](https://seadb-api.readme.io) for the exact route).
Only one SQL statement is executed per request, and the target base is part of
the request.

## Health check

`GET /ping` returns `{"ret":"pong"}` when the service is reachable.
