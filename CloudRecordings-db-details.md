# Apple Voice Memos SQLite Database

**Path:** `~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db`

This is a Core Data backed SQLite database (note the `Z_` prefixed columns and tables).

## Key Tables

| Table | Purpose |
|---|---|
| `ZCLOUDRECORDING` | Main recordings table |
| `ZRECORDING` | Local-only recordings (may be empty) |
| `ZFOLDER` | Folders for organizing recordings |

## ZCLOUDRECORDING Schema

```sql
CREATE TABLE ZCLOUDRECORDING (
  Z_PK INTEGER PRIMARY KEY,
  Z_ENT INTEGER,
  Z_OPT INTEGER,
  ZAUDIOFUTUREFLAGS INTEGER,
  ZFLAGS INTEGER,
  ZSHAREDFLAGS INTEGER,
  ZSILENCEREMOVERENABLED INTEGER,
  ZSKIPSILENCEENABLED INTEGER,
  ZSTUDIOMIXENABLED INTEGER,
  ZFOLDER INTEGER,              -- FK to ZFOLDER.Z_PK (NULL = no folder)
  ZDATE TIMESTAMP,              -- Core Data epoch (seconds since 2001-01-01)
  ZDURATION FLOAT,              -- Duration in seconds
  ZEVICTIONDATE TIMESTAMP,
  ZLOCALDURATION FLOAT,
  ZMTLAYERMIX FLOAT,
  ZPLAYBACKPOSITION FLOAT,
  ZPLAYBACKRATE FLOAT,
  ZPLAYBACKSPEED FLOAT,
  ZSTUDIOMIXLEVEL FLOAT,
  ZCUSTOMLABEL VARCHAR,         -- Timestamp label (ISO 8601)
  ZCUSTOMLABELFORSORTING VARCHAR,
  ZENCRYPTEDTITLE VARCHAR,      -- Recording title (plaintext despite the name)
  ZPATH VARCHAR,                -- Filename e.g. "20260127 132931-409ABD3B.m4a"
  ZUNIQUEID VARCHAR,
  ZAUDIOFUTUREUUIDS BLOB,
  ZAUDIODIGEST BLOB,
  ZAUDIOFUTURE BLOB,
  ZMTAUDIOFUTURE BLOB,
  ZVERSIONEDAUDIOFUTURE BLOB
);
```

### Notable Columns

| Column | Description |
|---|---|
| `ZENCRYPTEDTITLE` | Recording title (stored as plaintext despite the name) |
| `ZCUSTOMLABEL` | Timestamp label (ISO 8601) |
| `ZDATE` | Core Data timestamp (seconds since 2001-01-01) |
| `ZDURATION` | Duration in seconds |
| `ZPATH` | Filename (e.g., `20260127 132931-409ABD3B.m4a`) |
| `ZFOLDER` | FK to `ZFOLDER.Z_PK` (NULL = no folder) |
| `ZPLAYBACKPOSITION` / `ZPLAYBACKRATE` / `ZPLAYBACKSPEED` | Playback state |
| `ZSTUDIOMIXENABLED` / `ZSTUDIOMIXLEVEL` | Studio mix settings |
| `ZSKIPSILENCEENABLED` | Skip silence toggle |

## ZRECORDING Schema

```sql
CREATE TABLE ZRECORDING (
  Z_PK INTEGER PRIMARY KEY,
  Z_ENT INTEGER,
  Z_OPT INTEGER,
  ZITUNESPERSISTENTID INTEGER,
  ZLABELPRESET INTEGER,
  ZPENDINGRESTORE INTEGER,
  ZRECORDINGID INTEGER,
  ZSYNCED INTEGER,
  ZDATE TIMESTAMP,
  ZDURATION FLOAT,
  ZCUSTOMLABEL VARCHAR,
  ZPATH VARCHAR,
  ZUNIQUEID VARCHAR
);
```

## ZFOLDER Schema

```sql
CREATE TABLE ZFOLDER (
  Z_PK INTEGER PRIMARY KEY,
  Z_ENT INTEGER,
  Z_OPT INTEGER,
  ZCOUNTOFRECORDINGS INTEGER,   -- Auto-maintained via triggers
  ZRANK INTEGER,
  ZENCRYPTEDNAME VARCHAR,       -- Folder name (plaintext)
  ZUUID VARCHAR
);
```

## Useful Queries

### List recordings with human-readable dates

```sql
SELECT
  Z_PK,
  ZENCRYPTEDTITLE AS title,
  datetime(ZDATE + 978307200, 'unixepoch', 'localtime') AS date,
  printf('%.1f min', ZDURATION / 60.0) AS duration,
  ZPATH AS filename,
  ZFOLDER AS folder_id
FROM ZCLOUDRECORDING
ORDER BY ZDATE DESC;
```

### List recordings with folder names

```sql
SELECT
  cr.Z_PK,
  cr.ZENCRYPTEDTITLE AS title,
  datetime(cr.ZDATE + 978307200, 'unixepoch', 'localtime') AS date,
  printf('%.1f min', cr.ZDURATION / 60.0) AS duration,
  cr.ZPATH AS filename,
  f.ZENCRYPTEDNAME AS folder
FROM ZCLOUDRECORDING cr
LEFT JOIN ZFOLDER f ON cr.ZFOLDER = f.Z_PK
ORDER BY cr.ZDATE DESC;
```

### List folders

```sql
SELECT ZENCRYPTEDNAME AS name, ZCOUNTOFRECORDINGS AS recordings FROM ZFOLDER;
```

## Notes

- **Core Data epoch**: Dates are stored as seconds since 2001-01-01. Add `978307200` to convert to Unix epoch.
- **Audio files**: The `.m4a` files referenced by `ZPATH` are stored in the same `Recordings/` directory as the database.
- **CloudKit sync**: The `ANSC*` tables contain CloudKit sync metadata and are not needed for reading recording data.
- **"Encrypted" columns**: `ZENCRYPTEDTITLE` and `ZENCRYPTEDNAME` are stored as plaintext in the database despite their column names.
