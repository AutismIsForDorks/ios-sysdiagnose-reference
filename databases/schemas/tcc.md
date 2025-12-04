# TCC.db Schema Reference

Complete schema for the TCC (Transparency, Consent, and Control) database.

**Location**: `logs/Accessibility/TCC.db`

---

## Tables Overview

| Table | Purpose | Records |
|-------|---------|---------|
| `access` | Permission grants/denials | Many |
| `policies` | MDM policy definitions | Few |
| `active_policy` | Active policy assignments | Few |
| `access_overrides` | System-level overrides | Few |
| `expired` | Revoked permissions | Variable |
| `admin` | Admin key-value store | Few |
| `integrity_flag` | Integrity markers | Few |

---

## access Table

Primary table storing all permission decisions.

### Schema

```sql
CREATE TABLE access (
    service                           TEXT     NOT NULL,
    client                            TEXT     NOT NULL,
    client_type                       INTEGER  NOT NULL,
    auth_value                        INTEGER  NOT NULL,
    auth_reason                       INTEGER  NOT NULL,
    auth_version                      INTEGER  NOT NULL,
    csreq                             BLOB,
    policy_id                         INTEGER,
    indirect_object_identifier_type   INTEGER,
    indirect_object_identifier        TEXT     NOT NULL DEFAULT 'UNUSED',
    indirect_object_code_identity     BLOB,
    flags                             INTEGER,
    last_modified                     INTEGER  NOT NULL DEFAULT (CAST(strftime('%s','now') AS INTEGER)),
    pid                               INTEGER,
    pid_version                       INTEGER,
    boot_uuid                         TEXT     NOT NULL DEFAULT 'UNUSED',
    last_reminded                     INTEGER  NOT NULL DEFAULT (CAST(strftime('%s','now') AS INTEGER)),
    PRIMARY KEY (service, client, client_type, indirect_object_identifier),
    FOREIGN KEY (policy_id) REFERENCES policies(id) ON DELETE CASCADE ON UPDATE CASCADE
);
```

### Columns

| Column | Type | Description |
|--------|------|-------------|
| `service` | TEXT | Permission type (e.g., `kTCCServiceCamera`) |
| `client` | TEXT | App bundle ID or path |
| `client_type` | INTEGER | 0=bundle ID, 1=absolute path |
| `auth_value` | INTEGER | Authorization status |
| `auth_reason` | INTEGER | How permission was set |
| `auth_version` | INTEGER | Schema version |
| `csreq` | BLOB | Code signing requirement |
| `policy_id` | INTEGER | MDM policy reference |
| `indirect_object_identifier_type` | INTEGER | Type of indirect object |
| `indirect_object_identifier` | TEXT | Indirect object (e.g., for Photos access) |
| `indirect_object_code_identity` | BLOB | Indirect object code signing |
| `flags` | INTEGER | Additional flags |
| `last_modified` | INTEGER | Unix timestamp of last change |
| `pid` | INTEGER | Process ID that requested |
| `pid_version` | INTEGER | PID version |
| `boot_uuid` | TEXT | Boot session UUID |
| `last_reminded` | INTEGER | Last reminder timestamp |

### auth_value Values

| Value | Meaning |
|-------|---------|
| 0 | Denied |
| 1 | Unknown/Not determined |
| 2 | Allowed |
| 3 | Limited |

### auth_reason Values

| Value | Meaning |
|-------|---------|
| 1 | Error |
| 2 | User consent (prompt) |
| 3 | User set in Settings |
| 4 | System set |
| 5 | Service policy |
| 6 | MDM policy |
| 7 | Override policy |
| 8 | Missing usage description |
| 9 | Prompt timeout |
| 10 | Preflight unknown |
| 11 | Entitled |
| 12 | App type policy |

---

## policies Table

MDM-defined permission policies.

### Schema

```sql
CREATE TABLE policies (
    id        INTEGER  NOT NULL PRIMARY KEY,
    bundle_id TEXT     NOT NULL,
    uuid      TEXT     NOT NULL,
    display   TEXT     NOT NULL,
    UNIQUE (bundle_id, uuid)
);
```

### Columns

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER | Policy ID (auto-increment) |
| `bundle_id` | TEXT | Policy bundle identifier |
| `uuid` | TEXT | Policy UUID |
| `display` | TEXT | Display name |

---

## active_policy Table

Links clients to active policies.

### Schema

```sql
CREATE TABLE active_policy (
    client      TEXT    NOT NULL,
    client_type INTEGER NOT NULL,
    policy_id   INTEGER NOT NULL,
    PRIMARY KEY (client, client_type),
    FOREIGN KEY (policy_id) REFERENCES policies(id) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE INDEX active_policy_id ON active_policy(policy_id);
```

### Columns

| Column | Type | Description |
|--------|------|-------------|
| `client` | TEXT | Client identifier |
| `client_type` | INTEGER | Client type |
| `policy_id` | INTEGER | Referenced policy |

---

## access_overrides Table

System-level permission overrides.

### Schema

```sql
CREATE TABLE access_overrides (
    service TEXT NOT NULL PRIMARY KEY
);
```

### Columns

| Column | Type | Description |
|--------|------|-------------|
| `service` | TEXT | Service with system override |

---

## expired Table

Revoked or expired permissions.

### Schema

```sql
CREATE TABLE expired (
    service       TEXT    NOT NULL,
    client        TEXT    NOT NULL,
    client_type   INTEGER NOT NULL,
    csreq         BLOB,
    last_modified INTEGER NOT NULL,
    expired_at    INTEGER NOT NULL DEFAULT (CAST(strftime('%s','now') AS INTEGER)),
    PRIMARY KEY (service, client, client_type)
);
```

### Columns

| Column | Type | Description |
|--------|------|-------------|
| `service` | TEXT | Permission type |
| `client` | TEXT | Client identifier |
| `client_type` | INTEGER | Client type |
| `csreq` | BLOB | Code signing requirement |
| `last_modified` | INTEGER | When permission was last active |
| `expired_at` | INTEGER | When permission was revoked |

---

## admin Table

Administrative key-value storage.

### Schema

```sql
CREATE TABLE admin (
    key   TEXT    PRIMARY KEY NOT NULL,
    value INTEGER NOT NULL
);
```

---

## integrity_flag Table

Database integrity markers.

### Schema

```sql
CREATE TABLE integrity_flag (
    key   TEXT    PRIMARY KEY NOT NULL,
    value INTEGER NOT NULL
);
```

---

## Common Queries

### All Permissions with Details

```sql
SELECT
    service,
    client,
    CASE auth_value
        WHEN 0 THEN 'Denied'
        WHEN 1 THEN 'Unknown'
        WHEN 2 THEN 'Allowed'
        WHEN 3 THEN 'Limited'
    END as status,
    CASE auth_reason
        WHEN 2 THEN 'User Prompt'
        WHEN 3 THEN 'Settings'
        WHEN 4 THEN 'System'
        WHEN 6 THEN 'MDM'
    END as reason,
    datetime(last_modified, 'unixepoch', 'localtime') as modified
FROM access
ORDER BY last_modified DESC;
```

### Permission Timeline

```sql
SELECT
    datetime(last_modified, 'unixepoch', 'localtime') as when_changed,
    service,
    client,
    auth_value
FROM access
ORDER BY last_modified;
```

### Apps with Most Permissions

```sql
SELECT
    client,
    COUNT(*) as total,
    SUM(CASE WHEN auth_value = 2 THEN 1 ELSE 0 END) as allowed
FROM access
GROUP BY client
ORDER BY allowed DESC
LIMIT 20;
```

### Recently Expired Permissions

```sql
SELECT
    service,
    client,
    datetime(last_modified, 'unixepoch', 'localtime') as was_active,
    datetime(expired_at, 'unixepoch', 'localtime') as revoked
FROM expired
ORDER BY expired_at DESC
LIMIT 20;
```

---

## See Also

- [powerlog.md](powerlog.md) - PowerLog schema
- [../../privacy/tcc.md](../../privacy/tcc.md) - TCC analysis guide
