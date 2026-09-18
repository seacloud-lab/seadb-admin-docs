# Template Management

A **template** is a shared schema — a set of table, column, and index
definitions — that many tables can reuse. Tables that reference the same
template share a single copy of their metadata, so a change to the template
propagates to every table that uses it.

## Concepts

### Template base and template table

Each template is backed by a special **template base**, which stores only
metadata. Its **template tables** define columns, row schemas, and indexes but
hold no row or index data. A template base cannot have its tables deleted.

SeaDB provides every user with a default template base, addressed with the
reserved name `template`. `template` is therefore a reserved base name and
cannot be used for a regular base. Named templates can also be created, each
backed by its own template base.

### Table roles

A table in a regular base has one of three roles:

| Role | Description |
| --- | --- |
| Independent table | Not bound to a template; owns all of its metadata, row data, and indexes. |
| Template table | Lives in a template base; defines shared columns, row schema, and indexes but stores no row or index data. |
| Bound table | References a template table; its column and index definitions come from the template, while its row data and indexes are stored in the base that owns it. |

A bound table keeps its own table ID, name, row data, and indexes, and records
the referenced template base ID, template table ID, and template version. The
same template table can be referenced by tables with different names in one or
more bases, and a single base can reference template tables from different
template bases.

## Default template

Every user has a default template base. The reserved name `template` always
addresses the current user's default template, so you can pass it wherever the
CLI expects a base reference:

```bash
# Inspect the tables and columns of your default template.
seadb-cli base metadata template
seadb-cli base stats template
```

Change the default template by editing the table definitions in its template
base (see [Modifying a template](#modifying-a-template)).

## Creating and binding tables

When creating a table, you can leave it independent or bind it to a template by
specifying the template and template table:

- Without a template reference, the table is independent and maintains its own
  columns and indexes.
- With a template and a template table, the table is a bound table whose columns
  and indexes are read from the template table.

Refer to the [API reference](https://seadb-api.readme.io) for the endpoints that
create templates and bind tables to them.

## Bound table semantics

A bound table can be renamed independently, and can be created and deleted like
any table. Its column and index definitions, however, are controlled by the
template:

- Columns cannot be added, dropped, or renamed directly; they must be changed by
  modifying the referenced template table.
- Indexes cannot be created or dropped directly.
- Deleting a bound table removes only that table's own metadata, row data, and
  indexes — it does not affect the template table or other tables that reference
  it.
- A bound table can customize column data (for example, the options of a
  single-select or multiple-select column), which overrides the template table's
  value for that column.

## Modifying a template

A template is modified by editing the table definitions in its template base —
for example, adding or renaming a column, or changing an index. The change then
applies to every table bound to the template.

## Templates and import/export

When exporting and importing bases, the CLI preserves template bindings:

- Importing a base that uses a template reuses the same template when it exists.
- `seadb-cli import --skip-template` imports the base without using a template,
  converting a template-based base into an independent one.
- `seadb-cli import --table-templates <json>` maps tables to template tables and
  specifies which columns keep custom column data, allowing an independent base
  to be imported as a template-based one.

See the [SeaDB CLI](cli.md) for details.
