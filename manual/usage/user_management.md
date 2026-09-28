# User Management

SeaDB authenticates every request against a **user** and authorizes it against
the user's **roles** and **base ownership**. This page introduces the user and
role model.

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

A user can hold one or more roles.

## API key

An API key is a revocable credential issued to a user. It is a convenient
alternative to sending a username and password on every request, and it can be
deleted independently of the user account. Each user can create multiple API
keys, but one key per application scenario is recommended.

An API key is identified by a `key_id`. The full credential, `key_id:key`
Base64-URL-encoded, is returned only once at creation time as the `encoded`
value; the server stores only the hash of the key. Clients must save the
`encoded` value themselves.

## Base ownership

Each base is bound to exactly one owning user. The owner of a base can read and
write it; an `admin` can access any base. When a base is created, SeaDB records
the authenticated user as its owner.
