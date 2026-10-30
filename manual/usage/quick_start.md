# Quick Start

This guide walks through logging in as `root`, creating an ordinary user and
another administrator, and then creating a base and table to insert and query
data with SQL.

## Before you start

- Deploy SeaDB by following [Setup SeaDB](../setup/seadb_setup_single_node.md).
- Have [`seadb-cli`](https://sos-ch-dk-2.exo.io/cloud-seaticket-ai-seadb-cli/seadb-cli-x86_64-0.9) for Linux (x86_64) available on your client.


The examples connect to `http://127.0.0.1:8888`. Replace this address with your
SeaDB server address if it is running on another machine.

## 1. Log in as root

Configure the server address and log in:

```bash
seadb-cli set-config --server http://127.0.0.1:8888
seadb-cli login --username root
```

Enter the password for `root` when prompted. The password is not echoed to the
terminal. After a successful login, the CLI saves an API key and uses it
automatically for subsequent commands.

On headless Linux or over SSH, run
`seadb-cli set-config --credential-store file` before logging in if a system
keyring is unavailable. See [Credential storage](cli.md#credential-storage).

## 2. Create an ordinary user and another administrator

While logged in as `root`, create an ordinary user named `alice`:

```bash
seadb-cli user create alice --role default_role
```

Enter a password for `alice` when prompted. The `default_role` role allows
`alice` to read and write the bases she owns.

Create another administrator named `admin2`:

```bash
seadb-cli user create admin2 --role admin
```

Enter a password for `admin2` when prompted. The `admin` role allows this user
to manage users and all bases, and to read and write every base.

Check the user list:

```bash
seadb-cli user list
```

## 3. Log in as the ordinary user and create a base

Switch from `root` to `alice`:

```bash
seadb-cli logout
seadb-cli login --username alice
```

Enter the password you set for `alice`, then create a base:

```bash
seadb-cli base create --name quick-start
seadb-cli base list
```

The create command generates and prints the base UUID. Because `alice` creates
the base, she becomes its owner. Administrators, including `root` and `admin2`,
can also access it.

Set `BASE_ID` to the UUID printed by the create command for the remaining steps:

```bash
BASE_ID="<paste-the-base-uuid-here>"
```

## 4. Create a table

Create a `Tasks` table with a title, priority, and completion status:

```bash
seadb-cli sql --base "$BASE_ID" \
  -e "CREATE TABLE Tasks (title text, priority int, done bool);"
```

SeaDB automatically adds an auto-incrementing `_pk` column to identify each row.

## 5. Insert data with SQL

Insert two tasks:

```bash
seadb-cli sql --base "$BASE_ID" \
  -e "INSERT INTO Tasks (title, priority, done) VALUES ('Learn SeaDB', 1, false), ('Create a base', 2, true);"
```

The `_pk` values are generated automatically; do not include `_pk` in the
`INSERT` statement.

## 6. Query data with SQL

Read all tasks:

```bash
seadb-cli sql --base "$BASE_ID" \
  -e "SELECT _pk, title, priority, done FROM Tasks ORDER BY _pk;"
```

The CLI displays the two inserted tasks as a table. To find unfinished tasks:

```bash
seadb-cli sql --base "$BASE_ID" \
  -e "SELECT title, priority FROM Tasks WHERE done = false ORDER BY priority;"
```

This query returns the `Learn SeaDB` task with priority `1`.

Run each SQL statement in a separate `seadb-cli sql` command. Each call executes
one statement against the base specified by `--base`.

## Next steps

- [User Management](user_management.md): users, roles, API keys, and base ownership.
- [Base Management](base_management.md): tables, columns, indexes, and templates.
- [SQL Syntax](sql_syntax.md): supported SQL statements and data types.
- [SeaDB CLI](cli.md): configuration and command reference.
