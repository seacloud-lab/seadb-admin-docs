# SeaDB CLI

`seadb-cli` is a standalone Go command-line client for the SeaDB REST API. It
calls only existing SeaDB APIs; it does not access FoundationDB, the metabase,
or SeaDB's internal methods directly.

## Quick Start

Build, install, connect, and run your first query:

```bash
cd ./SeaDB

# Build the CLI into a temporary directory.
mkdir -p ../tmp
go build -o ../tmp/seadb-cli ./cmd/seadb-cli

# Install to $HOME/.local/bin and make it available on PATH.
../tmp/seadb-cli install
export PATH="$HOME/.local/bin:$PATH"

# Save the server address (verified with GET /ping).
seadb-cli set-config --server http://127.0.0.1:8888

# Log in interactively to create an API key.
seadb-cli login

# Create a base and run SQL against it.
seadb-cli base create
seadb-cli base create --name my-base
BASE_ID=f5a24a4e-a5bf-463a-b878-13b0a3a509c8
seadb-cli sql --base "$BASE_ID" -e "CREATE TABLE Tasks (title text, done bool);"
seadb-cli sql --base "$BASE_ID" -e "SELECT * FROM Tasks;"
```

The remaining sections describe each command in detail.

## Supported commands

The following subcommands are available:

- `set-config`: save the server address, timeout, and credential storage mode.
- `login`: log in and create an API key.
- `logout`: revoke the current API key and clean up local login state.
- `sql`: execute a single SQL statement against a base.
- `base`: create, list, inspect, delete bases, and change a base owner, ID, or name.
- `export`: export a base to a dump file.
- `import`: import a base from a dump file.
- `install`: install the current binary to the user directory.
- `uninstall`: remove the installed binary.

`help` and `completion` are provided by the command framework. The CLI runs one
command and then exits; there is no REPL or background process. Login state is
stored in the config file and the credential store, so terminals that use the
same config file share the login state.

User and API-key management commands are planned but not yet implemented; see
[Planned commands](#planned-commands).

## Prerequisites

- The default credential mode is `keyring`:
    - macOS uses Keychain.
    - Linux uses Secret Service, which requires an available user D-Bus session.
    - On headless Linux or over SSH, switch to `file` mode (see [Credential storage](#credential-storage)).

## Build and install

Run the following in the repository root:

```bash
cd ./SeaDB

# Run CLI unit tests. The tests use a local HTTP test server and do not
# depend on a running SeaDB or FoundationDB.
go test ./cmd/seadb-cli/...

# Build into a tmp directory next to the repository.
mkdir -p ../tmp
go build -o ../tmp/seadb-cli ./cmd/seadb-cli

# Verify the binary and its subcommands.
../tmp/seadb-cli --help
../tmp/seadb-cli -h
```

Optional cross-compilation examples:

```bash
# macOS Apple Silicon
CGO_ENABLED=0 GOOS=darwin GOARCH=arm64 \
  go build -o ../tmp/seadb-cli-darwin-arm64 ./cmd/seadb-cli

# Linux x86_64
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
  go build -o ../tmp/seadb-cli-linux-amd64 ./cmd/seadb-cli
```

Cross-compiled binaries can only run on the corresponding operating system and
CPU architecture.

### Install and upgrade

Use the built binary to install:

```bash
../tmp/seadb-cli install
```

This copies the current binary to:

```text
$HOME/.local/bin/seadb-cli
```

If `$HOME/.local/bin` is not on your `PATH`, the CLI prints the
`export PATH=...` line you need to run. It cannot modify the current terminal's
`PATH` or edit your shell configuration.

On macOS with zsh, add it to `PATH` permanently as follows:

```bash
printf '\nexport PATH="$HOME/.local/bin:$PATH"\n' >> ~/.zshrc
source ~/.zshrc
hash -r

command -v seadb-cli
seadb-cli --help
```

After rebuilding, overwrite the installed version with the source binary:

```bash
../tmp/seadb-cli install --force
../tmp/seadb-cli install -f
```

## Configuration file and login state

When `SEADB_CONFIG` is not set, the CLI uses the user-level default configuration:

```text
macOS: ~/Library/Application Support/seadb-cli/seadb_cli.yaml
Linux: ~/.config/seadb-cli/seadb_cli.yaml
```

When `XDG_CONFIG_HOME` is set on Linux, the file is at
`$XDG_CONFIG_HOME/seadb-cli/seadb_cli.yaml`. The
`cmd/seadb-cli/seadb_cli.yaml` file in the repository is not read automatically;
it is only used when `SEADB_CONFIG` is set explicitly.

The CLI creates the directory and config file automatically on the first
successful configuration or login, with mode `0600` on the config file. For
normal use, do not set `SEADB_CONFIG`, so that all terminals read the same
default config and share login state.

`SEADB_CONFIG` is only for temporary debugging, testing, or deliberately
isolating environments:

```bash
export SEADB_CONFIG="$HOME/.config/seadb-cli-dev.yaml"
```

The config file contains the following fields:

```yaml
server: http://127.0.0.1:8888
timeout: 30s
credential_store: keyring
```

- `server`: the SeaDB server address. It must use `http` or `https` and must not
  contain a username, query parameters, or a fragment.
- `timeout`: the HTTP request timeout. It defaults to `30s` when omitted; an
  explicit non-positive value such as `0s` is treated as invalid.
- `credential_store`: `keyring` or `file`. It defaults to `keyring` when
  omitted; an explicit empty value is invalid.
- `username` and `key_id` are maintained automatically by `login` and `logout`;
  do not set or modify them manually.

An empty file or a file containing only comments is equivalent to a missing
config file: all fields take their defaults and no parse error is raised.

## Commands

### set-config

```bash
seadb-cli set-config --server http://127.0.0.1:8888
seadb-cli set-config -s http://127.0.0.1:8888

# Change the timeout alone.
seadb-cli set-config --timeout 60s
seadb-cli set-config -t 60s

# Change the credential storage mode alone.
seadb-cli set-config --credential-store file
seadb-cli set-config -c file
```

When the server address is changed, the CLI first calls `GET /ping` and only
saves the new address if it receives `{"ret":"pong"}`.

`--server` is supported only by `set-config` and `login`; all other commands use
the address from the config file. To switch servers, run `seadb-cli logout`
first, then `set-config --server` or edit the config file directly.

`--timeout` is supported only by `set-config`, `login`, `logout`, and `sql`.
`set-config --timeout` writes the timeout to the config file; for the other
three commands, `--timeout` only overrides the timeout of the current request.
`install`, `uninstall`, `base`, `export`, and `import` do not accept the
`--timeout` parameter.

### login

```bash
seadb-cli login
```

Login reads a username and a non-echoing password and creates an API key with a
30-day validity. Login can also be scripted with explicit flags:

```bash
seadb-cli login --server http://127.0.0.1:8888 --username admin --password secret
```

The interactive flow looks like:

```text
Username: admin
Password:
OK: logged in as admin
```

After a successful login:

- The config file records `username` and `key_id`.
- The encoded API key is written to macOS Keychain or Linux Secret Service by
  default; the password is not saved.
- Subsequent commands use the API key automatically.

If no server address is configured yet, enter it interactively, or specify it
directly:

```bash
seadb-cli login --server http://127.0.0.1:8888
seadb-cli login -s http://127.0.0.1:8888
```

### logout

```bash
seadb-cli logout
```

Logout revokes the API key created by `login`, removes the credential from the
configured credential store, and removes `username` and `key_id` from the
configuration file. It is safe to run when the server already considers the key
invalid; the CLI still cleans up the local login state.

Use a one-off timeout when needed:

```bash
seadb-cli logout --timeout 60s
```

If the server cannot be reached or the credential cannot be read, the CLI
reports whether the server-side API key may still exist while cleaning up the
local state.

### sql

`sql` requires a target base UUID. The SQL can be passed with `-e` or read from
a pipe or redirected stdin:

```bash
BASE_ID=f5a24a4e-a5bf-463a-b878-13b0a3a509c8

seadb-cli sql --base "$BASE_ID" -e "SELECT * FROM t1"
seadb-cli sql -b "$BASE_ID" -e "SELECT * FROM t1"

printf 'UPDATE t1 SET age = 21 WHERE _pk = 1;\n' |
  seadb-cli sql --base "$BASE_ID"
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

Query results that include metadata are printed as a table. A write operation
with no result prints:

```text
OK: query executed
```

A query that returns an empty row set prints:

```text
OK: query returned no rows
```

Only one SQL statement is executed per call, and the target base is specified by
`--base`. Multi-statement scripts and SQL transaction commands are not
supported.

SQL statements are parsed by the server, and only the first statement in each
call is executed. When multiple statements are passed through a pipe, the rest
are ignored, so execute them one at a time.

Running `seadb-cli sql --base "$BASE_ID"` without `-e` in an interactive
terminal immediately returns `SQL statement is required via --execute/-e or
stdin`; it does not wait for terminal EOF. Use a pipe or redirection to read
from stdin.

### base

```bash
# Create a base; the CLI-generated UUID is printed on success.
seadb-cli base create
seadb-cli base create --name my-base

# List the bases owned by the current user.
seadb-cli base list
seadb-cli base list --scope mine

# Administrators can list all bases.
seadb-cli base list --scope all
seadb-cli base list -s all

# Inspect a single base.
seadb-cli base stats "$BASE_ID"
seadb-cli base metadata "$BASE_ID"

# Change the owner; administrators only.
seadb-cli base set-owner "$BASE_ID" alice

# Change a base ID or name; administrators only.
seadb-cli base set-id "$BASE_ID" <new-uuid>
seadb-cli base set-name "$BASE_ID" new-name

# Delete with interactive confirmation; use --yes/-y in scripts.
seadb-cli base delete "$BASE_ID"
seadb-cli base delete "$BASE_ID" --yes
```

- `base create` does not accept a user-supplied UUID; the CLI generates a UUID
  and prints it on success. `--name` gives the base an optional name.
- `base list` lists all bases owned by the currently authenticated user by
  default, equivalent to `--scope mine`.
- Regular users do not need to specify a scope. Administrators can use
  `--scope mine` to view their own bases and `--scope all` to view all bases.
- `base delete` asks for confirmation by default; non-interactive calls must
  pass `--yes` explicitly.
- `base set-owner`, `base set-id`, and `base set-name` are only available to
  administrators.
- `base stats` and `base metadata` print raw JSON; lists use table output.

#### template

The reserved name `template` addresses the current user's default template. Use
it wherever a base reference is expected to work with the default template:

```bash
seadb-cli base metadata template
seadb-cli base stats template
```

See [Base Management](base_management.md#templates) for how templates work.

### export

`export` writes one base to a dump file named after the base in the output
directory:

```bash
seadb-cli export --base "$BASE_ID" --output ./dump
seadb-cli export -b my-base -o ./dump
```

The dump file is written to `<output>/<base>.dump` and its path is printed on
success. The base reference may be a UUID or a name.

### import

`import` creates a new base from a dump file produced by `export`:

```bash
seadb-cli import --input ./dump/my-base.dump
seadb-cli import -i ./dump/my-base.dump
```

On success, the new base's UUID is printed. Optional flags:

- `--table-templates <json>` maps table names to template-table definitions
  during import.
- `--skip-template` imports without applying a base template.

`--skip-template` and `--table-templates` cannot be used together.

The `--table-templates` file is a JSON object keyed by table name. Each value
specifies the template to bind the table to and which columns keep their own
custom column data:

```json
{
    "Table1": {
        "template_table_name": "template_table1",
        "custom_columns": ["col1", "col2"]
    },
    "Table2": {
        "template_table_name": "template_table2",
        "custom_columns": []
    }
}
```

- `template_table_name` is the template table whose schema the table uses.
- `custom_columns` lists the columns that keep their own custom column data
  (for example, the options of a single-select or multiple-select column)
  instead of inheriting the template table's data.

A table not listed in the file is imported without a template binding. The
template table's schema must match the source table's schema, otherwise the
import is rejected.

### install and uninstall

See [Build and install](#build-and-install) for `install`. To uninstall, log out
first:

```bash
seadb-cli logout
seadb-cli uninstall
```

`uninstall` removes:

- `$HOME/.local/bin/seadb-cli`
- the default `seadb-cli` config directory and all of its contents, including
  `seadb_cli.yaml`, file credentials, custom configs, and backups.

If the default config is still logged in, log out with the default config first
to avoid leaving a server-side API key behind:

```bash
env -u SEADB_CONFIG seadb-cli logout
seadb-cli uninstall
```

A custom `SEADB_CONFIG` inside the default `seadb-cli` directory is removed
along with the directory; one outside that directory is not. The uninstall
command must be run by the installed `seadb-cli`, not by `../tmp/seadb-cli`.

## Planned commands

The following commands are part of the CLI design but have not been implemented
in the current `seadb-cli`.

### User management

```bash
seadb-cli user create <username>
seadb-cli user list
seadb-cli user set-password <username>
seadb-cli user set-role <username> --role admin
seadb-cli user delete <username>
```

- When creating a user or changing a password, the target password is read
  without echo by default.
- `--role` can be specified multiple times; the same role must not be repeated.
  Updating replaces all of the user's roles.
- Only the roles currently supported by SeaDB are accepted: `admin` and
  `default_role`.
- `user delete` uses the same confirmation rules as `base delete`.

### API key management

```text
seadb-cli api-key create <name>
seadb-cli api-key create <name> --expire-days 30
seadb-cli api-key create <name> --no-expire
seadb-cli api-key list
seadb-cli api-key delete <key-id> [<key-id>...]
```

- `api-key create` creates a key only for the currently authenticated user. The
  key expires after 30 days by default. `--expire-days` accepts a positive
  integer; a permanent key must be specified explicitly with `--no-expire`. The
  two parameters cannot be used together.
- On success, a single-row table shows `key_id`, `key_name`, `encoded`, the
  creation time, and the expiration time. `encoded` is returned by the server
  only once; the CLI does not write it to the login credential or config file.
- `api-key list` only lists the current user's keys; the output does not include
  `encoded`.
- Regular users can only delete their own keys; administrators can delete any
  user's key by key ID. Deletion asks for confirmation by default.
- Deleting the key currently used for login also removes the local login
  credential and login identity from the config file.

## Credential storage

The default `keyring` mode is suitable for macOS and Linux with a desktop
session. On headless Linux or over SSH, use `file` mode:

```bash
seadb-cli logout

seadb-cli set-config --credential-store file
seadb-cli set-config -c file

seadb-cli login
```

File mode stores the encoded API key in `<config file path>.credentials`. The
CLI creates the credential file with mode `0600`; when replacing an existing
file, its existing permissions are kept.

This file stores the encoded API key in plaintext, which is equivalent to a
password and is protected only by file permissions. `keyring` mode does not
leave a key file in the config directory.

You cannot change the server address or `credential_store` while logged in. To
switch servers or credential modes, run `logout` first, then `set-config`.

## Output conventions

- SQL queries and list commands use table output. Base statistics and metadata
  use raw JSON output.
- The CLI does not provide a selectable output format; JSON and CSV alternatives
  are not available for table-based commands.
- A successful response with data prints only the returned data, without extra
  status text.
- A successful response with no data prints:

    ```text
    OK: <operation result>
    ```

- Failures are printed to stderr:

    ```text
    Error: <reason>
    ```

- Exit codes: `0` on success, `1` on a request or business error, and `2` on a
  CLI argument error.

## Unsupported operations

- Managing users and API keys through `seadb-cli` is not yet implemented; see
  [Planned commands](#planned-commands).
- Managing users, grants, and bases through SQL statements such as `CREATE
  USER`, `GRANT`, or `CREATE DATABASE`.
- Creating, querying, or deleting custom roles.
- Authorizing one base to multiple users, or setting roles such as
  `reader`/`writer` on a base.
- User-facing JWT token management commands, and metrics, request, and
  cluster-node management commands.
- Other output formats such as JSON or CSV.
- Displaying DML affected rows; the current query API does not return this field.

## FAQ and exit codes

- `not logged in; run seadb-cli login`: run `seadb-cli login` in the
  environment of the current config file.
- `read login credential: ... does not match the configured server and
  username`: the local credential does not match the `server` or `username` in
  the current config. Run `seadb-cli logout`, then `seadb-cli login`.
- `invalid api key`: the server considers the current API key unusable. Run
  `seadb-cli logout`, then `seadb-cli login`, and retry the original command.
- `log out before changing the server` / `log out before changing the credential
  store`: you cannot change these two fields while logged in. Run
  `seadb-cli logout` first.
- `read login credential: ...; API key ... was not revoked`: `logout` cleaned up
  the local login state but could not revoke the server-side key. You can log in
  again directly; delete the old key on the server or wait for it to expire.
- `SeaDB reported API key ... as invalid, so it no longer needs to be revoked`:
  the server already considers the key invalid; local cleanup is enough for a
  successful logout.
- Only `logout` cleans up local login state. Business commands do not
  auto-clean when a key is invalid or the credential is corrupted. When
  `logout` cannot revoke a key, the output includes the key ID.
- macOS Keychain locked or denied access: unlock the login keychain in Keychain
  Access and allow access, then retry.
- Linux Secret Service unavailable: start a user D-Bus and Secret Service, or
  switch to `file` mode.
- A new terminal requires login again: check whether a different `SEADB_CONFIG`
  is set; only the same config path shares login state.

Exit codes:

- `0`: the command succeeded.
- `1`: a runtime error, such as a connection, authentication, credential, or
  server request failure.
- `2`: an incorrect command, argument, or argument value, including `help` and
  `completion` receiving an unknown command or extra arguments. An unknown
  command returns `2` even with `--help` and prints no help; for a valid command
  name, `--help` takes precedence, prints help, and returns `0`.
