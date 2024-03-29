```
  ____                   ____  ____
 / ___|___ _ __ ___  ___|  _ \| __ )
| |   / _ \ '__/ _ \/ __| | | |  _ \
| |__|  __/ | |  __/\__ \ |_| | |_) |
 \____\___|_|  \___||___/____/|____/
```

# About

CeresDB is a database system designed to allow for the storage and retrieval of semi-structured data. I.e. one that conforms to a "top-level schema" where columns types are known, but those columns can in-turn contain dictionaries or lists.

## Naming

CeresDB is named after the Roman goddess of agriculture for the way in which the system "harvests" data from the files it is stored on

# Running CeresDB

## Running Locally

To run CeresDB locally, ensure that the `ceresdb` binary is in your path and configure your instance using the information in [Configuring CeresDB](#configuring-ceresdb)
Afterwards, just run the `ceresdb` binary and your instance will start.

## Running in Docker

To run CeresDB via Docker, pull down the CeresDB image with

```
docker pull jfcarter2358/ceresdb:<desired tag>
```

then run with

```
docker run -p 7437:7437 jfcarter2358/ceresdb:<desired tag>
```

Adding environment variables as desired to configure your instance as described in [Configuring CeresDB](#configuring-ceresdb)

# Querying a CeresDB Instance

CeresDB uses the Antler Query Language (AQL) to interact with the data contained within the database. This language is made up of 9 main actions that can act on 5 different resources:

## COLLECTION

Collections act as logical groupings of data with the same schema within a database. They are composed of multiple records

> Details on Schema dictionaries can be found in the [Schema Format](#schema-format) section

### DELETE

Deletes a collection from a database

```
DELETE COLLECTION <name of database>.<name of collection>
```

### GET

Returns the collections contained in a database

```
GET COLLECTION <name of database>
```

### POST

Creates a new collection

```
POST COLLECTION <name of database>.<name of collection> <dict of schema>
```

### PUT

Update the a collection's schema

```
POST COLLECTION <name of database>.<name of collection> <dict of schema>
```

## DATABASE

Databases act as the highest-level grouping of data which can contain multiple collections

### DELETE

Deletes a database

```
DELETE DATABASE <name of database>
```

### GET

Returns the databases contained in the CeresDB instance

```
GET DATABASE
```

### POST

Creates a new database

```
POST DATABASE <name of database>
```

## PERMIT

Permits control access to databases and are made up of records containing usernames within the instance and their roles

> Details on access roles can be found in the [Access Roles](#access-roles) section

### DELETE

Deletes a permit

```
DELETE PERMIT <name of database> <id or list of ids of permit to delete or use '-' to delete ids from piped input>
```

### GET

Returns the permits contained in a CeresDB database

```
GET PERMIT <name of database> <fields to include in output or use '*' to include all>
```

### POST

Creates a new permit

```
POST PERMIT <name of database> <dict of permit with format {"username":"<username to add>","role":"<access role to add>"}>
```

### PUT

Overwrites a permit with new data

```
PUT PERMIT <name of database> <id or list of ids to overwrite> <dict or list of dicts of data to update to>
```
## RECORD

Records are the items of data inserted/retrieved from the collections within a database

### DELETE

Deletes a record

```
DELETE RECORD <name of database>.<name of collection> <id or list of ids of permit to delete or use '-' to delete ids from piped input>
```

### GET

Returns the databases contained in the CeresDB instance

```
GET RECORD <name of database>.<name of collection> <fields to include in output or use '*' to include all>
```

### POST

Creates a new database

```
POST DATABASE <name of database>.<name of collection> <dict or list of dicts of data to insert>
```

### PATCH

Updates a field in multiple records

```
PATCH RECORD <name of database>.<name of collection> <id or list of ids to update> <dict of fields to update and their new values>
```

### PUT

Overwrites a record with new data

```
PUT RECORD <name of database>.<name of collection> <id or list of ids to overwrite> <dict or list of dicts of data to update to>
```

## USER

Users control who can access the database and what their instance-level permissions are

> Details on access roles can be found in the [Access Roles](#access-roles) section

### DELETE

Deletes a user

```
DELETE USER <id or list of ids of permit to delete or use '-' to delete ids from piped input>
```

### GET

Returns the users contained in a CeresDB instance

```
GET USER <fields to include in output or use '*' to include all>
```

### POST

Creates a new user

```
POST USER <dict of permit with format {"username":"<username to add>","role":"<access role to add>","password":"<password for the user to authenticate with>"}>
```

### PUT

Overwrites a user with new data

```
PUT USER <id or list of ids to overwrite> <dict or list of dicts of data to update to>
```

## Modifier Actions

While CeresDB uses `DELETE`, `GET`, `PATCH`, `POST`, and `PUT` to manipulate data, the results of these actions can then be piped into others to perform complex actions. The modifier actions that output can be piped into (in addition to piping the output into statements which take the `-` as an argument described above) are:

### COUNT

Returns the number of items returned from the input query in the format `{"count":"<number of items>"}`

```
<Other query> | COUNT
```

### FILTER

> NOTE: Filter can only be used on GET queries for records, users, and permits

Allows you to filter out the results of a `GET` query using logical expressions made up of `<field name> <comparison operator> <value>` joined together via logical operators

```
<Other query> | FILTER <field name> <comparison operator> <value> <logical operator> ...
```

**Comparison Operators**

- `>` Greater than
- `>=` Greater than or equal to
- `=` Equal to
- `<` Less than
- `<=` Less than or equal to
- `!=` Not equal to

**Logical Operators**

- `AND` And
- `OR` Or
- `NOT` Not
- `XOR` Exclusive or

### LIMIT

Allows you to reduce the number of results to a specified maximum

```
<Other query> | LIMIT <maximum desired number of items>
```

### ORDERASC

Orders results in ascending order by a specified key

```
<Other query> | ORDERASC <key to order by>
```

### ORDERDSC

Orders results in descending order by a specified key

```
<Other query> | ORDERDSC <key to order by>
```

# Schema Format

Schemas take the form of a dictionary with each key being a field in the corresponding records that will be held in the collection and each value being the datatype of said field. Allowed datatypes are:

- `STRING`
- `INT`
- `FLOAT`
- `BOOL`
- `LIST` (not searchable by filters or able to be ordered)
- `DICT` (not searchable by filters or able to be ordered)
- `ANY` (not searchable by filters or able to be ordered)

# Access Roles

CeresDB supports three main access roles which are:

- `READ` allows the user to read data
- `WRITE` allows the user to read, write, overwrite, and delete data
- `ADMIN` allows the user to manage users or permits

The required access mapping for each action is shown below

- `COLLECTION`
    - `DELETE` - database level `ADMIN`
    - `GET` - database level `READ`, `WRITE`, or `ADMIN`
    - `POST` - database level `ADMIN`
    - `PUT` - database level `ADMIN`
- `DATABASE`
    - `DELETE` - instance level `WRITE` or `ADMIN`
    - `GET` - instance level `READ`, `WRITE`, or `ADMIN`
    - `POST` - instance level `WRITE` or `ADMIN`
- `PERMIT`
    - `DELETE` - database level `ADMIN`
    - `GET` - database level `READ`, `WRITE`, or `ADMIN`
    - `POST` - database level `ADMIN`
    - `PUT` - database level `ADMIN`
- `RECORD`
    - `DELETE` - database level `WRITE` or `ADMIN`
    - `GET` - database level `READ`, `WRITE`, or `ADMIN`
    - `POST` - database level `WRITE` or `ADMIN`
    - `PATCH` - database level `WRITE` or `ADMIN`
    - `PUT` - database level `WRITE` or `ADMIN`
- `USER`
    - `DELETE` - instance level `ADMIN`
    - `GET` - instance level `READ`, `WRITE`, or `ADMIN`
    - `POST` - instance level `ADMIN`
    - `PUT` - instance level `ADMIN`

# Configuring CeresDB

CeresDB can be configured either by providing a configuration JSON file of the following format:

```
{
    "log-level": "info",
    "home-dir": ".ceresdb",
    "data-dir": ".ceresdb/data",
    "index-dir": ".ceresdb/indices",
    "storage-line-limit": 16384,
    "port": 7437
}
```

and then setting the environment variable `CERESDB_CONFIG_PATH` to point to said JSON file. Alternatively, you can use environment variables to configure your Ceres instance with the following variables:

- `CERESDB_LOG_LEVEL`
- `CERESDB_HOME_DIR`
- `CERESDB_DATA_DIR`
- `CERESDB_INDEX_DIR`
- `CERESDB_STORAGE_LINE_LIMIT`
- `CERESDB_PORT`
- `CERESDB_DEFAULT_ADMIN_PASSWORD`

# To Do

**v1.0.0**

- [x] Basic reading from structure
- [x] Basic writing to structure
- [x] Basic deletion from structure
- [x] AQL query parsing
    - [x] Basic query parsing
    - [x] Nested conditionals with parenthesis
- [x] Basic indices
- [x] Collection schema
    - [x] Schema definition on collection creation
    - [x] Schema modification
- [x] Query parsing/response
    - [x] COUNT
        - [x] COLLECTION
        - [x] DATABASE
        - [x] PERMIT
        - [x] RECORD
        - [x] USER
    - [x] DELETE
        - [x] COLLECTION
        - [x] DATABASE
        - [x] PERMIT
        - [x] RECORD
        - [x] USER
    - [x] FILTER
    - [x] GET
        - [x] COLLECTION
        - [x] DATABASE
        - [x] PERMIT
        - [x] RECORD
        - [x] USER
    - [x] LIMIT
    - [x] ORDERASC
    - [x] ORDERDSC
    - [x] PATCH
        - [x] RECORD
    - [x] POST
        - [x] COLLECTION
        - [x] DATABASE
        - [x] PERMIT
        - [x] RECORD
        - [x] USER
    - [x] PUT
        - [x] COLLECTION
        - [x] PERMIT
        - [x] RECORD
        - [x] USER
- [x] UDP server
- [x] Authentication
    - [x] User management
    - [x] User roles
- [x] Break free space out into separate package
- [x] Validation
    - [x] Validate schema values
    - [x] Validate data values against schema
- [x] Direct access protection
    - [x] _auth database
    - [x] _user collections
- [x] Concurrent connection support via read/write queue
- [x] Automated tests
    - [x] Regression
    - [x] Stress
- [x] Write documentation
    - [x] README
    - [x] Readthedocs

**v1.1.0**
- [x] `jq` data manipulation support
- [x] Switch to using HTTP for server
- [x] Leader/follower DB replication

**v1.2.0**
- [ ] Concurrent data extraction

**v1.3.0**

- [ ] JSON dot notation field selection
- [ ] Better Index management
    - [ ] Multi-word string indices
    - [ ] List indices
    - [ ] Dict indices
- [ ] Improve unit test coverage
- [ ] New datatypes
    - [ ] Date
    - [ ] Datetime
    - [ ] Time
- [ ] Defined constants
    - [ ] Now
- [ ] Filter mathematical operations
    - [ ] `+`
    - [ ] `-`
    - [ ] `*`
    - [ ] `/`
    - [ ] `^`
- [ ] Token authentication

**v1.4.0**

- [ ] Better support for concurrency

# Contact

This software is written by John Carter. If you have any questions or concerns feel free to create an issue on GitHub or send me an email at jfcarter2358(at)gmail.com
