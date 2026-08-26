# SeaDB CLI: build and usage

This page describes the complete process of building, installing, configuring, logging in, executing SQL, and managing bases with `seadb-cli`.

The following subcommands are currently supported:

- `set-config`: save the server address, timeout, and credential storage mode.
- `login`: log in interactively and create an API key.
- `logout`: revoke the current API key and clean up local login state.
- `sql`: execute a single SQL statement against a base.
- `base`: create, list, inspect, delete bases, and change a base owner.
- `install`: install the current binary to the user directory.
- `uninstall`: remove the installed binary.

`help` and `completion` are provided by the command framework. The CLI runs one command and then exits; there is no REPL or background process. Login state is stored in the config file and the credential store, so terminals that use the same config file share the login state.

## Prerequisites

- The default credential mode is `keyring`:
    - macOS uses Keychain.
    - Linux uses Secret Service, which requires an available user D-Bus session.
    - On headless Linux or over SSH, switch to `file` mode (see [Credential storage](#credential-storage)).

## Build and test

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

Cross-compiled binaries can only run on the corresponding operating system and CPU architecture.

## Install and upgrade

Use the built binary to install:

```bash
../tmp/seadb-cli install
```

This copies the current binary to:

```text
$HOME/.local/bin/seadb-cli
```

If `$HOME/.local/bin` is not on your `PATH`, the CLI prints the `export PATH=...` line you need to run. It cannot modify the current terminal's `PATH` or edit your shell configuration.

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

When `XDG_CONFIG_HOME` is set on Linux, the file is at `$XDG_CONFIG_HOME/seadb-cli/seadb_cli.yaml`. The `cmd/seadb-cli/seadb_cli.yaml` file in the repository is not read automatically; it is only used when `SEADB_CONFIG` is set explicitly.

The CLI creates the directory and config file automatically on the first successful configuration or login. For normal use, do not set `SEADB_CONFIG`, so that all terminals read the same default config and share login state.

`SEADB_CONFIG` is only for temporary debugging, testing, or deliberately isolating environments:

```bash
export SEADB_CONFIG="$HOME/.config/seadb-cli-dev.yaml"
```

The config file contains the following fields:

```yaml
server: http://127.0.0.1:7777
timeout: 30s
credential_store: keyring
```

- `server`: the SeaDB server address. It must use `http` or `https` and must not contain a username, query parameters, or a fragment.
- `timeout`: the HTTP request timeout. It defaults to `30s` when omitted; an explicit non-positive value such as `0s` is treated as invalid.
- `credential_store`: `keyring` or `file`. It defaults to `keyring` when omitted; an explicit empty value is invalid.
- `username` and `key_id` are maintained automatically by `login` and `logout`; do not set or modify them manually.

An empty file or a file containing only comments is equivalent to a missing config file: all fields take their defaults and no parse error is raised.

## First-time setup

### Save the connection configuration

```bash
seadb-cli set-config --server http://127.0.0.1:7777
seadb-cli set-config -s http://127.0.0.1:7777

# Change the timeout alone.
seadb-cli set-config --timeout 60s
seadb-cli set-config -t 60s
```

When the server address is changed, the CLI first calls `GET /ping` and only saves the new address if it receives `{"ret":"pong"}`.

`--server` is supported only by `set-config` and `login`; all other commands use the address from the config file. To switch servers, run `seadb-cli logout` first, then `set-config --server` or edit the config file directly.

`--timeout` is supported only by `set-config`, `login`, `logout`, and `sql`. `set-config --timeout` writes the timeout to the config file; for the other three commands, `--timeout` only overrides the timeout of the current request. `install` and `uninstall` do not send requests and reject this parameter.

### Log in

```bash
seadb-cli login
```

Login must run in an interactive terminal. The CLI reads a username and a non-echoing password and creates an API key with a 30-day validity:

```text
Username: admin
Password:
OK: logged in as admin
```

After a successful login:

- The config file records `username` and `key_id`.
- The encoded API key is written to macOS Keychain or Linux Secret Service by default; the password is not saved.
- Subsequent commands use the API key automatically.

If no server address is configured yet, enter it interactively, or specify it directly:

```bash
seadb-cli login --server http://127.0.0.1:7777
seadb-cli login -s http://127.0.0.1:7777
```

Login cannot run in a non-interactive terminal; complete login in an interactive terminal first.

### Execute SQL

`sql` requires a target base UUID. The SQL can be passed with `-e` or read from a pipe or redirected stdin:

```bash
BASE_ID=f5a24a4e-a5bf-463a-b878-13b0a3a509c8

seadb-cli sql --base "$BASE_ID" -e "SELECT * FROM t1"
seadb-cli sql -b "$BASE_ID" -e "SELECT * FROM t1"

printf 'UPDATE t1 SET age = 21 WHERE _pk = 1;\n' |
  seadb-cli sql --base "$BASE_ID"
```

Query results that include metadata are printed as a table. A write operation with no result prints:

```text
OK: query executed
```

A query that returns an empty row set prints:

```text
OK: query returned no rows
```

SQL statements are parsed by the server, and only the first statement in each call is executed. When multiple statements are passed through a pipe, the rest are ignored, so execute them one at a time.

Running `seadb-cli sql --base "$BASE_ID"` without `-e` in an interactive terminal immediately returns `SQL statement is required via -e or stdin`; it does not wait for terminal EOF. Use a pipe or redirection to read from stdin.

### Base management

```bash
# Create a base; the CLI-generated UUID is printed on success.
seadb-cli base create

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

# Delete with interactive confirmation; use --yes/-y in scripts.
seadb-cli base delete "$BASE_ID"
seadb-cli base delete "$BASE_ID" --yes
```

`base stats` and `base metadata` print raw JSON; lists use table output. The base UUID is generated by the CLI, and permission and business validation are handled by the server.

## Credential storage

The default `keyring` mode is suitable for macOS and Linux with a desktop session. On headless Linux or over SSH, use `file` mode:

```bash
seadb-cli logout

seadb-cli set-config --credential-store file
seadb-cli set-config -c file

seadb-cli login
```

File mode stores the encoded API key in `<config file path>.credentials`. The CLI creates the credential file with mode `0600`; when replacing an existing file, its existing permissions are kept.

This file stores the encoded API key in plaintext, which is equivalent to a password and is protected only by file permissions. `keyring` mode does not have this problem — it does not leave a key file in the config directory.

You cannot change the server address or `credential_store` while logged in. To switch servers or credential modes, run `logout` first, then `set-config`.

## Uninstall

Log out before uninstalling:

```bash
seadb-cli logout
seadb-cli uninstall
```

`uninstall` removes:

- `$HOME/.local/bin/seadb-cli`
- the default `seadb-cli` config directory and all of its contents, including `seadb_cli.yaml`, file credentials, custom configs, and backups.

If the default config is still logged in, log out with the default config first to avoid leaving a server-side API key behind:

```bash
env -u SEADB_CONFIG seadb-cli logout
seadb-cli uninstall
```

A custom `SEADB_CONFIG` inside the default `seadb-cli` directory is removed along with the directory; one outside that directory is not. The uninstall command must be run by the installed `seadb-cli`, not by `../tmp/seadb-cli`.

## FAQ and exit codes

- `not logged in; run seadb-cli login`: run `seadb-cli login` in the environment of the current config file.
- `read login credential: ... does not match the configured server and username`: the local credential does not match the `server` or `username` in the current config. Run `seadb-cli logout`, then `seadb-cli login`.
- `invalid api key`: the server considers the current API key unusable. Run `seadb-cli logout`, then `seadb-cli login`, and retry the original command.
- `log out before changing the server` / `log out before changing the credential store`: you cannot change these two fields while logged in. Run `seadb-cli logout` first.
- `read login credential: ...; API key ... was not revoked`: `logout` cleaned up the local login state but could not revoke the server-side key. You can log in again directly; delete the old key on the server or wait for it to expire.
- `SeaDB reported API key ... as invalid, so it no longer needs to be revoked`: the server already considers the key invalid; local cleanup is enough for a successful logout.
- Only `logout` cleans up local login state. Business commands do not auto-clean when a key is invalid or the credential is corrupted. When `logout` cannot revoke a key, the output includes the key ID.
- macOS Keychain locked or denied access: unlock the login keychain in Keychain Access and allow access, then retry.
- Linux Secret Service unavailable: start a user D-Bus and Secret Service, or switch to `file` mode.
- A new terminal requires login again: check whether a different `SEADB_CONFIG` is set; only the same config path shares login state.

Exit codes:

- `0`: the command succeeded.
- `1`: a runtime error, such as a connection, authentication, credential, or server request failure.
- `2`: an incorrect command, argument, or argument value — including `help` and `completion` receiving an unknown command or extra arguments.
