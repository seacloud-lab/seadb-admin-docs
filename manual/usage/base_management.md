# Base Management

A **base** is SeaDB's unit of data organization, roughly equivalent to a
database or namespace. Every SQL statement and data operation targets a specific
base. This page describes bases, tables, columns, and indexes, and how to manage
them.

## Concepts

### Base

Each base has a UUID that uniquely identifies it. A base can also be given an
optional human-readable **name**; once a base has a name, the name can be used in
place of the UUID.

Every base is owned by exactly one user (see
[User Management](user_management.md)). The owner and any `admin` can access the
base.

### Table

A base contains one or more tables. A table is a collection of rows sharing a
schema.

### Column

Each table has columns with a SeaDB column type. Every table also has a built-in
`_pk` column — an auto-incrementing integer that uniquely identifies each row.
Supported column types:

| Type | Description |
| --- | --- |
| `text` | Text up to 100 KB. |
| `int64` | 64-bit integer. |
| `float32` | 32-bit floating point. |
| `float64` | 64-bit floating point. |
| `bool` | Boolean. |
| `datetime` | Date/time with optional timezone. |
| `single-select` | Single-select column with option metadata. |
| `multiple-select` | Multiple-select column with option metadata. |
| `list` | List of elements of the same type. |

### Index

Indexes speed up queries. SeaDB does not create column indexes automatically;
create them explicitly with `CREATE INDEX`. The `_pk` column already acts as a
built-in unique index.

## Create and manage bases

A base is created with a generated UUID, and optionally given a name:

```bash
# Create a base; the CLI generates and prints the UUID.
seadb-cli base create
seadb-cli base create --name my-base
```

List the bases you own, or — as an administrator — all bases:

```bash
seadb-cli base list
seadb-cli base list --scope all
```

Inspect a base's statistics or full metadata:

```bash
seadb-cli base stats <base>
seadb-cli base metadata <base>
```

Administrators can rename a base, change its ID, or transfer ownership; the
owner or an administrator can delete a base:

```bash
seadb-cli base set-name <base> new-name
seadb-cli base set-id <base> <new-uuid>
seadb-cli base set-owner <base> alice
seadb-cli base delete <base>
```

See the [SeaDB CLI](cli.md) for the full command reference.

## Define tables and columns with SQL

Tables and columns are defined with SQL DDL:

```sql
CREATE TABLE [IF NOT EXISTS] table_name [(column_name data_type, ...)];
DROP TABLE [IF EXISTS] table_name;
ALTER TABLE table_name RENAME TO new_table_name;
ALTER TABLE table_name ADD COLUMN [IF NOT EXISTS] column_name data_type;
ALTER TABLE table_name DROP COLUMN [IF EXISTS] column_name;
ALTER TABLE table_name RENAME COLUMN old_column_name TO new_column_name;
```

Example:

```sql
CREATE TABLE Tasks (title text, priority int, done bool);
ALTER TABLE Tasks ADD COLUMN assignee text;
DROP TABLE Tasks;
```

Run DDL through the CLI against a specific base:

```bash
seadb-cli sql --base <base> -e "CREATE TABLE Tasks (title text, done bool);"
```

## Manage indexes with SQL

```sql
CREATE [UNIQUE] INDEX [IF NOT EXISTS] index_name ON table_name (column_name, ...);
DROP INDEX [IF EXISTS] index_name;
DROP INDEX [IF EXISTS] 'index_id';
```

Index names must be unique within a base. `CREATE INDEX` builds the index in the
background; the `_pk` column is a built-in unique index and cannot be created or
dropped with DDL.

Example:

```sql
CREATE INDEX idx_priority ON Tasks (priority);
CREATE UNIQUE INDEX idx_title ON Tasks (title);
DROP INDEX idx_priority;
```

## Run SQL

All data operations are executed against a specific base. See
[SQL Syntax](sql_syntax.md) for the supported SQL statements and the
[SeaDB CLI](cli.md) for running them.

## Templates

A **template** is a reusable set of table definitions. Tables that use the same
template share its column and index definitions, so changing the template
applies to all of them.

Every user has a default template. Use the reserved name `template` wherever a
base reference is expected to work with it. The CLI accepts `template` in the
`base` commands, so you can list the tables of your default template with:

```bash
seadb-cli base metadata template
seadb-cli base stats template
```

Create and modify template tables the same way as regular tables, by targeting
`template` — for example with `POST /api/v1/template/tables` (see the
[API reference](https://seadb-api.readme.io)).

When exporting and importing bases, the CLI preserves template bindings:

- Importing a base that uses a template reuses the same template when it exists.
- `seadb-cli import --skip-template` imports the base without a template.
- `seadb-cli import --table-templates <json>` maps tables to template tables and
  specifies which columns keep custom column data.

See the [SeaDB CLI](cli.md) for details.
