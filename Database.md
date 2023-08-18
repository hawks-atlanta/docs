# Database

This document contains the models present in the database.

## Authentication

### Tables

```mermaid
erDiagram
	users {
		UUID      uuid          "PRIMARY KEY NOT NULL"
		TIMESTAMP create_at     "NOT NULL DEFAULT CURRENT_TIMESTAMP"
		TIMESTAMP modified_at   "NOT NULL DEFAULT CURRENT_TIMESTAMP"
		STRING    username      "UNIQUE NOT NULL"
		STRING    password_hash "NOT NULL"
	}
	
	logins  {
		UUID      uuid       "PRIMARY KEY NOT NULL"
		TIMESTAMP event_at   "NOT NULL"
		UUID      user_uuid  "FOREIGN KEY `user` . `uuid`"
		STRING    ip_address "NOT NULL"
	}
```

### Relations

```mermaid
erDiagram
	users ||--o{ logins: "Has many"
```

### Queries

#### Create user

```sql
INSERT INTO users (uuid, username, password_hash) 
VALUES (?, ?, ?)
```

#### Authenticate

```sql
SELECT uuid
FROM users
WHERE 
	Username     = ?
	AND password = ?
LIMIT 1

-- - Then if result UUID is not null

INSERT INTO logins (uuid, event_at, user_uuid, ip_address) VALUES (?, ?, ?, ?)
```

#### Update password

```sql
UPDATE users
SET
	modified_at   = NOW()
	password_hash = ?
WHERE
	uuid              = ?
	AND password_hash = ?
```

## Metadata

### Tables

```mermaid
erDiagram
	archives {
		UUID    uuid        "PRIMARY KEY"
		STRING  hash_sum    "NOT NULL"
		UINT    size        "NOT NULL"
		BOOLEAN ready       "DEFAULT FALSE"
	}

	files {
		UUID    uuid            "PRIMARY KEY"
		UUID    owner_uuid      "INDEX; NOT NULL"
		UUID    parent_uuid     "DEFAULT NULL; FOREIGN KEY files.uuid"
		UUID    archive_uuid    "DEFAULT NULL; FOREIGN KEY archives.uuid"
		STRING  volume          "DEFAULT NULL"
		STRING  name            "INDEX; NOT NULL; UNIQUE (owner_id, parent_id, name)"
	}
```

#### Tables description

|   Name    | Description                                                  |
| :-------: | :----------------------------------------------------------- |
|  `files`  | This table contains metadata about the files and directories of the users. Note that, as in the UNIX file system, directories are also a type of file. |
| `archive` | This table contains metadata about the archives (Actual files stored in the storage system). |

#### Relations

```mermaid
erDiagram
	files ||--o| archives: "Has zero or one"
```

#### Design notes

- If the `files.parent_uuid` value is `NULL`, then the file is in the user's root directory.
- If the `files.archive_uuid` value is `NULL`, then the file is a directory.
- If the `files.volume` value is `NULL`, then the file wasn't stored yet. 
- It's not necessary to store the `files.backup_volume` value because it can be inferred from the `files.volume` value, E.g. if the `files.volume` value is `volume_1`, then the `files.backup_volume` value is `volume_1_backup`.
- The `files.name` value needs to have a 3-tuple unique constraint so that the same **user** can't have two files with the same **name** in the same **directory**.

### Queries

#### Create directory

```sql
INSERT OR IGNORE INTO files(owner_uuid, parent_uuid, name)
VALUES(?, ?, ?)
```

#### Create file

First, create the `archive` metadata: 

```sql
INSERT OR IGNORE INTO archives(hash_sum, size)
VALUES(?, ?)
```

Then, create the `file` metadata:

```sql
INSERT OR IGNORE INTO files(owner_uuid, parent_uuid, archive_uuid, name)
VALUES(?, ?, ?, ?)
```

Note that the `archives.ready` value is set to `FALSE` by default and the `files.volume` value is set to `NULL` by default because the file wasn't stored at this point.

#### Nark as ready

First, update the `archive` metadata:

```sql
UPDATE archives
SET
	ready = TRUE
WHERE
	uuid = ?
```

Then, update the `file` metadata:

```sql
UPDATE files
SET
	volume = ?
WHERE
	archive_uuid = ?
```

#### Delete file

First, delete the `archive` metadata. Note that, if the file is a directory, then the `archive_uuid` value is `NULL`, so the `DELETE` query should not delete anything:

```sql
DELETE FROM archives
WHERE 
	archives.uuid = (
		SELECT archive_uuid
		FROM files
		WHERE
			files.uuid = ?
	)
```

Then, delete the `file` metadata:

```sql
DELETE FROM files
WHERE
	files.uuid = ?
```

#### List directory

```sql
SELECT uuid, name, archive_uuid 
FROM files
WHERE
	owner_uuid  = ?
	AND parent_uuid = ?
LIMIT ? OFFSET (? - 1)
```

#### Get file UUID

```sql
SELECT uuid
FROM files
WHERE
	owner_uuid	 	= ?
	AND name   		= ?
```

## Worker

### Structure

```
/
├── files/
│   ├── [VOLUME_MOUNT_DATA_1]/
│   │   └── [FILE_UUID_X]
│   └── [VOLUME_MOUNT_DATA_2]/
│       └── [FILE_UUID_Y]
└── backups/
    ├── [VOLUME_MOUNT_BACKUP_1]/
    │   └── [FILE_UUID_X]
    └── [VOLUME_MOUNT_BACKUP_2]/
        └── [FILE_UUID_Y]
```

|   Path    |                         Description                          |
| :-------: | :----------------------------------------------------------: |
|  `files`  | This directory is a volume mounted on the pod. Used as main file storage point |
| `backups` | In this directory are mounted other volumes used as backup points |