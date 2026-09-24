# SQL Syntax

SeaDB supports basic DDL, `SELECT`, `INSERT`, `REPLACE`, `UPDATE`, `DELETE`,
and basic `JOIN` statements.

## Query statements

The `SELECT` statement syntax:

```text
SELECT [DISTINCT] fields FROM table_name [WhereClause] [GroupByClause] [HavingClause] [OrderByClause] [Limit Option]
```

The `JOIN` statement syntax:

```text
SELECT ... FROM table1, table2 WHERE table1.column1 = table2.column2 AND ...

SELECT ... FROM table1 [LEFT/INNER] JOIN table2 ON table1.column1 = table2.column2 AND ...
```

`JOIN` queries have the following restrictions:

1. Without the `JOIN` keyword:
    - Tables in the `FROM` clause cannot be repeated.
    - Every table in the `FROM` clause must be associated with at least one JOIN
      condition.
    - JOIN conditions go in the `WHERE` clause and are joined with `AND`.
    - When the two columns have the same type, the JOIN condition can only be an
      equality comparison, for example `table1.column1 = table2.column2`.
    - When the two columns have different types — one a list column and one a
      scalar column whose type matches the list element type — the JOIN
      condition can use `IN`, for example `table1.column IN table2.list_column`.
2. With the `JOIN` keyword:
    - `INNER JOIN` and `LEFT JOIN` are supported; `FULL JOIN` and other JOIN
      forms are not.
    - JOIN conditions go in the `ON` clause.
    - When the two columns have different types — one a list column and one a
      scalar column whose type matches the list element type — the JOIN
      condition can use `IN`, for example `table1.column IN table2.list_column`.

Notes:

- The `WHERE` clause supports most expressions (arithmetic, comparison, and so
  on), including the keywords `[NOT] LIKE`, `IN`, `BETWEEN ... AND ...`, `AND`,
  `OR`, `NOT`, `IS [NOT] TRUE`, and `IS [NOT] NULL`.
- `GROUP BY` is strict: apart from the aggregate functions (`COUNT`, `SUM`,
  `MAX`, `MIN`, `AVG`), every selected field must also appear in the `GROUP BY`
  clause.
- `HAVING` filters rows after `GROUP BY` aggregation. Only fields in the
  `GROUP BY` clause or aggregate functions can be referenced in `HAVING`; its
  syntax is otherwise the same as `WHERE`.
- `ORDER BY` sorts by a field, which must appear in the `SELECT` expression.
  For example, `SELECT a FROM table ORDER BY b` is invalid, while
  `SELECT a FROM table ORDER BY a` works.
- `LIMIT` uses MySQL syntax: `LIMIT ... OFFSET ...`.
- `AS` aliases a returned field. For example, `SELECT table.a AS a FROM table`
  returns the column under the name `a`.

## Data modification statements

The `INSERT`, `REPLACE`, `UPDATE`, and `DELETE` statement syntax:

```text
INSERT INTO table_name [column_list] VALUES value_list [, ...]

REPLACE INTO table_name [column_list] VALUES value_list [, ...]

UPDATE table_name SET column_name = value [, ...] [WhereClause]

DELETE FROM table_name [WhereClause]
```

- `column_list` is a parenthesized, comma-separated list of column names. When
  omitted, it defaults to all updatable columns.
- `value_list` is a parenthesized, comma-separated list of values that must
  correspond one-to-one with the names in `column_list`, for example
  `(1, "2", 3.0)`.
- Multi-value columns (such as multiple-select) wrap the value list in
  parentheses or brackets, for example
  `(1, "2", 3.0, ("foo", "bar"), [1.0, 2.0])`.
- Single-select and multiple-select columns use option names, not option keys.
- `WhereClause` is an optional `WHERE` clause; when omitted, all rows match.

Note: the `_pk` column does not support `INSERT`, `REPLACE`, and `UPDATE`.

## Data types

The following table maps SeaDB column data to the types used in SQL fields.

| SeaDB data type | SQL field type | Return format | where/having | group by/order by |
| --- | --- | --- | --- | --- |
| Text | String |  | Supported | Supported |
| Long text | String |  | Supported | Supported |
| 32-bit float | Float32 |  | Supported | Supported |
| 64-bit float | Float64 |  | Supported | Supported |
| Integer | Int64 |  | Supported | Supported |
| Single select | String | Returns the option key by default; set the request's `convert_keys` parameter to `true` to return the option name. | Constants use the option name, for example `WHERE single_select = "New York"`. | Sorted by option definition order. |
| Multiple select | List of string | Returns the option keys by default; set the request's `convert_keys` parameter to `true` to return the option names. | Constants use the option name. See the list type section below for matching rules. | Supported; see the list type section below. |
| Checkbox | Bool |  | Supported | Supported |
| Date | Datetime | Returns an RFC 3339 string. | Constants use ISO format time strings such as `"2006-1-2"` and `"2006-1-2 15:04:05"`. RFC 3339 strings such as `"2020-12-31T23:59:60Z"` are also supported. | Supported |
| List | List of elements of the same type |  | Supported | Supported; see the list type section below. |

## List type handling

When a list-typed column is used in a `WHERE` condition, the behavior depends on
the operator. Operators not listed below are unsupported.

| Operator | Rule |
| --- | --- |
| `IN`, list extended syntax (for example `has any of`) | Handled by the operator's rules. |
| `=`, `!=` | Compares the list elements in order. |
| `IS NULL` | `NULL` when the column has no data or an empty list. |
| `IS TRUE` | Always `false`. |

In `GROUP BY` and `ORDER BY`, lists are ordered as follows:

- Elements are compared one by one from the first; the list with the smaller
  element comes first.
- If all compared elements are equal, the shorter list comes first.
- If the lengths are also equal, the two lists are equal.

Among aggregate functions (`min`, `max`, `sum`, `avg`), only `min` and `max`
support list columns; `sum` and `avg` are unsupported.

## NULL values

`NULL` differs from `0`; it represents an empty value. The following values are
treated as `NULL`:

- An empty cell in a table.
- A value that cannot be converted to the column's type.
- An empty string (`""`). This differs from standard SQL.
- A list value, according to the rules in the [list type](#list-type-handling)
  section.

In `WHERE` conditions:

- An arithmetic operation with a `NULL` operand yields `NULL`.
- `!=`, `NOT LIKE`, `NOT IN`, `NOT BETWEEN`, `HAS NONE OF`, `IS NOT TRUE`, and
  `IS NULL` yield `TRUE` when they encounter `NULL`.
- `AND`, `OR`, and `NOT` treat `NULL` as `FALSE`.
- Aggregate functions (`min`, `max`, `sum`, `avg`) ignore `NULL`.

In `JOIN` matching:

- `NULL` does not participate in equality matching. If either side of a JOIN
  condition is `NULL` (including when both sides are `NULL`), the row is not
  treated as a match.

## Data definition statements

The DDL statement syntax:

```text
Tables and columns:
CREATE TABLE [IF NOT EXISTS] table_name [(column_name data_type, ...)];
DROP TABLE [IF EXISTS] table_name;
ALTER TABLE table_name RENAME TO new_table_name;
ALTER TABLE table_name ADD COLUMN [IF NOT EXISTS] column_name data_type;
ALTER TABLE table_name DROP COLUMN [IF EXISTS] column_name;
ALTER TABLE table_name RENAME COLUMN old_column_name TO new_column_name;

Indexes:
CREATE [UNIQUE] INDEX [IF NOT EXISTS] index_name ON table_name (column_name, ...);
DROP INDEX [IF EXISTS] index_name;
DROP INDEX [IF EXISTS] 'index_id';
```

Tables and columns:

- `CREATE TABLE` creates an empty table or a table with one or more columns.
  A created or renamed table name must not contain `*`.
- `ALTER TABLE ... ADD COLUMN` adds one column at a time.
- `IF NOT EXISTS` applies to `CREATE TABLE`, `ADD COLUMN`, and `CREATE INDEX`;
  no error is raised if the target already exists.
- `IF EXISTS` applies to `DROP TABLE`, `DROP COLUMN`, and `DROP INDEX`; no error
  is raised if the target does not exist.
- DDL cannot be executed inside a SQL transaction.

Indexes:

- An index name is unique within a base.
- `CREATE INDEX` builds the index in the background and supports one or more
  index columns. `UNIQUE` creates a unique index.
- When an index ID is numeric, write it as a string in `DROP INDEX`, for
  example `DROP INDEX '0005';`.
- `_pk` is a system built-in unique index and cannot be created or dropped with
  DDL.

DDL keywords and column type names are reserved words. When a table, column, or
index name conflicts with a reserved word, quote it with backticks, for example
`CREATE TABLE example (\`date\` DATE);` and `SELECT \`date\` FROM example;`.

Unsupported DDL:

- Column constraints: `PRIMARY KEY`, `NOT NULL`, `DEFAULT`.
- Changing a column type: `ALTER TABLE ... ALTER COLUMN ... TYPE`.
- Column types without a SeaDB equivalent, such as `NUMERIC`, `UUID`, `JSON`,
  `BYTEA`, `TIME`, and `INTERVAL`.

## Extended syntax

### List extended syntax

Some SeaDB column types support extended syntax, including `HAS ANY OF`,
`HAS ALL OF`, `HAS NONE OF`, and `IS EXACTLY`. For example, to query all rows
whose multiple-select column `city` contains both `"New York"` and `"Paris"`:

```sql
SELECT * FROM table WHERE city HAS ANY OF ("New York", "Paris");
```

The parenthesized list is equivalent to an `IN` clause.

## Indexes

SeaDB supports creating indexes on rows to speed up queries.

SeaDB also supports unique indexes on a column, in which case the column values
must be unique.

The `_pk` column can be used directly as a unique index, so it does not need an
additional index.

When you add a column to a table, its index is not created automatically; create
it explicitly through the API. When you delete a column, any index on that
column is deleted automatically.
