# SeaDB CLI overview

SeaDB provides a REST API for SQL execution, base management, user management, and base ownership. The `seadb-cli` tool is a standalone Go command-line client for this API. The CLI only calls existing SeaDB APIs; it does not access FoundationDB, the metabase, or SeaDB's internal methods directly.

See [Build and usage](usage.md) for how to build, install, and configure the CLI.

## Commands

### SQL

```bash
seadb-cli sql --base <uuid> -e '<sql>'
echo '<sql>' | seadb-cli sql --base <uuid>
```

Examples:

```bash
# DQL
seadb-cli sql --base 2b5d1e7a-5ca1-4e61-b236-c4f0c50cf65b \
  -e 'SELECT title, priority FROM Tasks WHERE done = false LIMIT 10;'

# DML
seadb-cli sql --base 2b5d1e7a-5ca1-4e61-b236-c4f0c50cf65b \
  -e 'UPDATE Tasks SET done = true WHERE _pk = 1;'

# DDL via stdin
echo 'CREATE TABLE Tasks (title text, priority int, done bool);' |
  seadb-cli sql --base 2b5d1e7a-5ca1-4e61-b236-c4f0c50cf65b
```

The following statements are supported:

- DQL: `SELECT`, `EXPLAIN`
- DML: `INSERT`, `REPLACE`, `UPDATE`, `DELETE`
- DDL: `CREATE TABLE`, `DROP TABLE`, `ALTER TABLE`, `CREATE INDEX`, `DROP INDEX`

Only one SQL statement is executed per call, and the target base is specified by `--base`. Multi-statement scripts and SQL transaction commands are not supported.

### Base management

```bash
seadb-cli base create
seadb-cli base list
seadb-cli base list --scope mine
seadb-cli base list --scope all
seadb-cli base stats <uuid>
seadb-cli base metadata <uuid>
seadb-cli base delete <uuid>
seadb-cli base set-owner <uuid> <username>
```

- `base create` does not accept a user-supplied UUID; the CLI generates a standard UUID and prints it on success.
- `base list` lists all bases owned by the current user by default, equivalent to `--scope mine`.
- Regular users do not need to specify a scope. Administrators can use `--scope mine` to view their own bases and `--scope all` to view all bases.
- `base delete` asks for confirmation by default; non-interactive calls must pass `--yes` explicitly.
- `base set-owner` is only available to administrators.

### User management

```bash
seadb-cli user create <username>
seadb-cli user list
seadb-cli user set-password <username>
seadb-cli user set-role <username> --role admin
seadb-cli user delete <username>
```

- When creating a user or changing a password, the target password is read without echo by default.
- `--role` can be specified multiple times; the same role must not be repeated. Updating replaces all of the user's roles.
- Only the roles currently supported by SeaDB are accepted: `admin` and `default_role`.
- `user delete` uses the same confirmation rules as `base delete`.

### API key management

```text
seadb-cli api-key create <name>
seadb-cli api-key create <name> --expire-days 30
seadb-cli api-key create <name> --no-expire
seadb-cli api-key list
seadb-cli api-key delete <key-id> [<key-id>...]
```

- `api-key create` only creates a key for the current logged-in user, with a default validity of 30 days. `--expire-days` accepts a positive integer; a permanent key must be specified explicitly with `--no-expire`. The two parameters cannot be used together.
- On success, a single-row table shows `key_id`, `key_name`, `encoded`, the creation time, and the expiration time. `encoded` is returned by the server only once; the CLI does not write it to the login credential or config file.
- `api-key list` only lists the current user's keys; the output does not include `encoded`.
- Regular users can only delete their own keys; administrators can delete any user's key by key ID. Deletion asks for confirmation by default.
- Deleting the key currently used for login also removes the local login credential and login identity from the config file.

## Output conventions

- SQL queries, base lists, user lists, and API key lists use table output. Base statistics and metadata use JSON output.
- Only the `table` format is supported; `JSON` and `CSV` are not provided.
- A successful response with data only prints the data, without extra status text.
- A successful response with no data prints:

    ```text
    OK: <operation result>
    ```

- Failures are printed to stderr:

    ```text
    Error: <reason>
    ```

- Exit codes: `0` on success, `1` on a request or business error, and `2` on a CLI argument error.

## Unsupported operations

- Managing users, grants, and bases through SQL statements such as `CREATE USER`, `GRANT`, or `CREATE DATABASE`.
- Creating, querying, or deleting custom roles.
- Authorizing one base to multiple users, or setting roles such as `reader`/`writer` on a base.
- User-facing JWT token management commands, and metrics, request, and cluster-node management commands.
- Other output formats such as JSON or CSV.
- Displaying DML affected rows; the current query API does not return this field.

## CLI ↔ API mapping

| CLI command | SeaDB API |
| --- | --- |
| `login` / `set-config --server` connection check | `GET /ping` |
| `login` | `POST /api/v1/users/api_keys` |
| `logout` | `DELETE /api/v1/users/api_keys` |
| `sql` | `POST /api/v1/{base_id}/query` |
| `base create` | `POST /api/v1/{base_id}/base` |
| `base list` / `--scope mine` | `GET /api/v1/users/bases` |
| `base list --scope all` | `GET /api/v1/bases` |
| `base stats` | `GET /api/v1/{base_id}/base-info` |
| `base metadata` | `GET /api/v1/{base_id}/metadata` |
| `base delete` | `DELETE /api/v1/{base_id}/base` |
| `base set-owner` | `POST /api/v1/{base_id}/base/update-base-owner` |
| `user create` | `POST /api/v1/users` |
| `user list` | `GET /api/v1/users` |
| `user set-password` / `user set-role` | `PUT /api/v1/users` |
| `user delete` | `DELETE /api/v1/users` |
| `api-key create` | `POST /api/v1/users/api_keys` |
| `api-key list` | `GET /api/v1/users/api_keys` |
| `api-key delete` | `DELETE /api/v1/users/api_keys` |

## Cluster mode prerequisites

For the CLI to fully support management commands in cluster mode, the Proxy must forward the following endpoints:

```text
GET    /ping
GET    /api/v1/users/bases
POST   /api/v1/{base_id}/base/update-base-owner
POST   /api/v1/users/api_keys
DELETE /api/v1/users/api_keys
GET    /api/v1/users/api_keys
GET    /api/v1/users
POST   /api/v1/users
PUT    /api/v1/users
DELETE /api/v1/users
```
