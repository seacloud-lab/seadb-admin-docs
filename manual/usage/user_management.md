# User Management

SeaDB uses **users**, **roles**, and **base ownership** to control access. This
page describes the user model and how administrators manage accounts with
`seadb-cli`. See the [SeaDB CLI user reference](cli.md#user) for command syntax
and output details.

## User

A user is an identity that can access SeaDB. Each user has a unique `username`
and a `password`. Passwords are stored as `argon2id` hashes, never in plaintext.

## Role

Roles control what a user is allowed to do. SeaDB ships with two built-in roles
and does not yet support custom roles:

| Role | Description |
| --- | --- |
| `admin` | Manages the users, and all bases. Can read and write every base. |
| `default_role` | Reads and writes only the bases it owns. This is the role assigned to ordinary application users. |

A user can hold one or more roles. New users receive `default_role` unless
roles are explicitly specified. A user with `admin` has administrator privileges
whether or not it also holds `default_role`.

Only administrators can create, list, update, or delete users. This includes
password changes: ordinary users cannot use the user management commands to
change their own passwords.

## Create and manage users

### Log in as an administrator

The first administrator is created during server setup; see
[Create the first administrator](../setup/seadb_setup_single_node.md#4-create-the-first-administrator).
After the server is running, log in with that account to manage other users:

```bash
seadb-cli login --username seadb-admin
```

Enter the administrator's password when prompted. Subsequent commands use the
API key created by `login`.

### Create a user

Specify a unique username and enter the new user's password at the non-echoing
prompt:

```bash
seadb-cli user create alice
```

The user receives `default_role` and can create and manage its own bases. To
create another administrator, specify the role explicitly:

```bash
seadb-cli user create bob --role admin
```

For non-interactive use, supply the target user's password with `--pwd` or `-p`:

```bash
seadb-cli user create alice --pwd '<password>' --role default_role
```

Creation prints the username and assigned roles as a table. Passwords are not
included in the output.

### List users

```bash
seadb-cli user list
```

The CLI retrieves all pages of users and prints their usernames and roles.

### Change a password

An administrator can set a new password for any user, including its own account:

```bash
seadb-cli user set-password alice
seadb-cli user set-password alice --pwd '<new-password>'
```

This changes only the password, leaving the user's roles unchanged. Changing a
password does not revoke existing API keys; revoke unwanted keys separately
through the API.

### Replace a user's roles

```bash
# Give Alice administrator privileges.
seadb-cli user set-role alice --role admin

# Return Alice to an ordinary user.
seadb-cli user set-role alice --role default_role
```

The supplied roles replace the user's entire role list rather than adding to
it. At least one role is required. Only `admin` and `default_role` are supported;
to assign both, repeat the option:

```bash
seadb-cli user set-role alice --role admin --role default_role
```

Each role may be specified only once. Changing roles does not change the user's
password or transfer ownership of any bases.

### Delete a user

```bash
seadb-cli user delete alice
```

The CLI asks for confirmation. Enter `y` or `yes` to proceed; the default is to
cancel. Non-interactive calls must explicitly pass `--yes` or `-y`:

```bash
seadb-cli user delete alice --yes
```

Deletion removes the account, its role assignments, API key records, base
ownership records, and template records. It does not delete the data in the
user's bases. Transfer bases to another existing user before deleting the
account if they should retain an owner:

```bash
seadb-cli base set-owner <base> bob
seadb-cli user delete alice
```

See [Base Management](base_management.md) for managing bases and templates.

## API key

An API key is a revocable credential issued to a user. It is a convenient
alternative to sending a username and password on every request, and it can be
deleted independently of the user account. Each user can create multiple API
keys, but one key per application scenario is recommended.

An API key is identified by a `key_id`. The full credential, `key_id:key`
Base64-URL-encoded, is returned only once at creation time as the `encoded`
value; the server stores only the hash of the key. Clients must save the
`encoded` value themselves.

API keys authenticate requests as the user that created them; they do not carry
an independent role list. Authorization uses that user's roles and base
ownership. Use the credential in the HTTP header:

```text
Authorization: ApiKey <encoded>
```

Users can create and list their own keys and delete their own keys.
Administrators can also delete another user's key by its ID. These operations
are available through the [API Reference](https://seadb-api.readme.io).
The CLI's `login` creates a 30-day API key and stores it in the configured
credential store; `logout` revokes that key and clears local login state. See
[SeaDB CLI](cli.md#login) for details.

## Base ownership

Each base is bound to exactly one owning user. The owner of a base can read and
write it; an `admin` can access any base. When a base is created, SeaDB records
the authenticated user as its owner.

An administrator can transfer ownership with
`seadb-cli base set-owner <base> <username>`. The new owner must already exist.
SeaDB does not support assigning multiple owners or reader/writer roles to a
base. Users and roles are managed through the CLI or REST API, not SQL
statements such as `CREATE USER` or `GRANT`.
