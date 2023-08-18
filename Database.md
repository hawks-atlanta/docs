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

### Relations

```mermaid
erDiagram
	files ||--o| archives: "Has zero or one"
```

### Queries

#### Create directory

```sql
INSERT INTO directories (owner_id, volume, parent_id, name)
VALUES (?, ?, ?, ?)
```

#### Create file

```sql
INSERT OR IGNORE INTO files(owner_id, name, size, hashsum)
VALUES(?, ?, ?, ?)
```

#### Make ready 

First, create a new location (directory) for the file (as needed). Note that this operation should be performed twice to create the main and replica locations.

```sql
INSERT OR IGNORE INTO directories(owner_id, volume, parent_id, name)
VALUES(?, ?, ?, ?)
```

Then, update the file with the new locations and set it as ready.

```sql
UPDATE files
SET
	main_location_id    = ?
	replica_location_id = ?
	ready               = TRUE
WHERE
	uuid                = ?
```

#### Delete file

```sql
DELETE FROM files
WHERE
	owner_id	 = ?
	AND uuid   = ?
```

#### Delete directory

First, delete all files in the directory.

```sql
DELETE FROM files
WHERE
	owner_id	 							= ?
	AND main_location_id    = ?
```

Then, delete all subdirectories.

```sql
DELETE FROM directories
WHERE
	owner_id	 = ?
	AND parent_id = ?
```

Finally, delete the directory itself.

```sql
DELETE FROM directories
WHERE
	owner_id	 = ?
	AND uuid   = ?
```

#### List directory

List files: 

```sql
SELECT uuid, name
FROM files
WHERE
	owner_id				= ?
	AND main_location_id = ?
LIMIT ? OFFSET (? - 1)
```

List subdirectories:

```sql
SELECT uuid, name
FROM directories
WHERE
	owner_id	 = ?
	AND parent_id = ?
```

#### Get file UUID

```sql
SELECT uuid
FROM files
WHERE
	owner_id	 = ?
	AND name   = ?
```

## Worker

### Structure

```
/
├── files:[VOLUME_MOUNT_DATA]/
│   ├── [FILE_UUID_X]
│   └── [FILE_UUID_Y]
└── backups/
    ├── [VOLUME_MOUNT_BACKUP_1]/
    │   ├── [FILE_UUID_X]
    │   └── [FILE_UUID_Y]
    └── [VOLUME_MOUNT_BACKUP_2]/
        ├── [FILE_UUID_X]
        └── [FILE_UUID_Y]
```

|   Path    |                         Description                          |
| :-------: | :----------------------------------------------------------: |
|  `files`  | This directory is a volume mounted on the pod. Used as main file storage point |
| `backups` | In this directory are mounted other volumes used as backup points |