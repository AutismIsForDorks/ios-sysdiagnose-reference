# TCC.db - Transparency, Consent, and Control Database

The TCC database (`logs/Accessibility/TCC.db`) stores all privacy permission decisions on iOS. This is one of the most forensically valuable artifacts for understanding what data apps can access.

## Location

```
logs/Accessibility/TCC.db
```

## Database Schema

### access Table (Primary)

Stores all permission grants/denials:

```sql
CREATE TABLE access (
    service        TEXT        NOT NULL,     -- Permission type
    client         TEXT        NOT NULL,     -- App bundle ID or path
    client_type    INTEGER     NOT NULL,     -- 0=bundle ID, 1=path
    auth_value     INTEGER     NOT NULL,     -- Authorization status
    auth_reason    INTEGER     NOT NULL,     -- How permission was set
    auth_version   INTEGER     NOT NULL,     -- Schema version
    csreq          BLOB,                     -- Code signing requirement
    policy_id      INTEGER,                  -- MDM policy reference
    indirect_object_identifier_type INTEGER,
    indirect_object_identifier TEXT NOT NULL DEFAULT 'UNUSED',
    indirect_object_code_identity BLOB,
    flags          INTEGER,
    last_modified  INTEGER     NOT NULL DEFAULT (CAST(strftime('%s','now') AS INTEGER)),
    pid            INTEGER,                  -- Process that requested
    pid_version    INTEGER,
    boot_uuid      TEXT        NOT NULL DEFAULT 'UNUSED',
    last_reminded  INTEGER     NOT NULL DEFAULT (CAST(strftime('%s','now') AS INTEGER)),
    PRIMARY KEY (service, client, client_type, indirect_object_identifier)
);
```

### policies Table

MDM-managed policies:

```sql
CREATE TABLE policies (
    id          INTEGER NOT NULL PRIMARY KEY,
    bundle_id   TEXT    NOT NULL,
    uuid        TEXT    NOT NULL,
    display     TEXT    NOT NULL,
    UNIQUE (bundle_id, uuid)
);
```

### active_policy Table

Currently active policies:

```sql
CREATE TABLE active_policy (
    client       TEXT    NOT NULL,
    client_type  INTEGER NOT NULL,
    policy_id    INTEGER NOT NULL,
    PRIMARY KEY (client, client_type),
    FOREIGN KEY (policy_id) REFERENCES policies(id)
);
```

### access_overrides Table

System-level permission overrides:

```sql
CREATE TABLE access_overrides (
    service TEXT NOT NULL PRIMARY KEY
);
```

### expired Table

Revoked/expired permissions:

```sql
CREATE TABLE expired (
    service        TEXT        NOT NULL,
    client         TEXT        NOT NULL,
    client_type    INTEGER     NOT NULL,
    csreq          BLOB,
    last_modified  INTEGER     NOT NULL,
    expired_at     INTEGER     NOT NULL DEFAULT (CAST(strftime('%s','now') AS INTEGER)),
    PRIMARY KEY (service, client, client_type)
);
```

## Service Types (Permissions)

### Location

| Service | Description |
|---------|-------------|
| `kTCCServiceLocation` | Location access |
| `kTCCServiceLocationAlways` | Background location |
| `kTCCServiceLocationWhenInUse` | Foreground location |

### Contacts & Calendar

| Service | Description |
|---------|-------------|
| `kTCCServiceAddressBook` | Contacts access |
| `kTCCServiceCalendar` | Calendar access |
| `kTCCServiceReminders` | Reminders access |

### Media

| Service | Description |
|---------|-------------|
| `kTCCServicePhotos` | Full photo library |
| `kTCCServicePhotosAdd` | Add-only photo access |
| `kTCCServiceCamera` | Camera access |
| `kTCCServiceMicrophone` | Microphone access |
| `kTCCServiceMediaLibrary` | Apple Music/Media |

### Sensors

| Service | Description |
|---------|-------------|
| `kTCCServiceMotion` | Motion & Fitness |
| `kTCCServiceWillow` | Health data |
| `kTCCServiceFocusStatus` | Focus/DND status |

### Communications

| Service | Description |
|---------|-------------|
| `kTCCServiceBluetoothAlways` | Bluetooth access |
| `kTCCServiceBluetoothPeripheral` | Bluetooth peripheral |
| `kTCCServiceBluetoothWhileInUse` | Foreground Bluetooth |

### Privacy (iOS 18+)

| Service | Description |
|---------|-------------|
| `kTCCServiceUserTracking` | App Tracking Transparency |
| `kTCCServiceSiri` | Siri integration |
| `kTCCServiceSpeechRecognition` | Speech recognition |
| `kTCCServiceFaceID` | Face ID usage |

## Authorization Values

| Value | Meaning |
|-------|---------|
| `0` | Denied |
| `1` | Unknown/Not determined |
| `2` | Allowed |
| `3` | Limited (e.g., selected photos) |

## Auth Reason Codes

| Code | Meaning |
|------|---------|
| `1` | Error |
| `2` | User consent (prompted) |
| `3` | User set in Settings |
| `4` | System set |
| `5` | Service policy |
| `6` | MDM policy |
| `7` | Override policy |
| `8` | Missing usage description |
| `9` | Prompt timeout |
| `10` | Preflight unknown |
| `11` | Entitled |
| `12` | App type policy |

## Query Examples

### List All Permissions

```sql
SELECT service, client,
       CASE auth_value
           WHEN 0 THEN 'Denied'
           WHEN 2 THEN 'Allowed'
           WHEN 3 THEN 'Limited'
           ELSE 'Unknown'
       END as status,
       datetime(last_modified, 'unixepoch') as modified
FROM access
ORDER BY last_modified DESC;
```

### Find Apps with Location Access

```sql
SELECT client, auth_value, datetime(last_modified, 'unixepoch')
FROM access
WHERE service LIKE '%Location%' AND auth_value = 2;
```

### Find Recently Modified Permissions

```sql
SELECT service, client, auth_value,
       datetime(last_modified, 'unixepoch') as modified
FROM access
WHERE last_modified > strftime('%s', 'now', '-7 days')
ORDER BY last_modified DESC;
```

### Count Permissions by App

```sql
SELECT client,
       COUNT(*) as total_permissions,
       SUM(CASE WHEN auth_value = 2 THEN 1 ELSE 0 END) as allowed
FROM access
GROUP BY client
ORDER BY allowed DESC;
```

### Find Denied Permissions

```sql
SELECT service, client, datetime(last_modified, 'unixepoch')
FROM access
WHERE auth_value = 0
ORDER BY last_modified DESC;
```

### Find Expired Permissions

```sql
SELECT service, client,
       datetime(last_modified, 'unixepoch') as modified,
       datetime(expired_at, 'unixepoch') as expired
FROM expired
ORDER BY expired_at DESC;
```

## Bash Commands

### Quick Permission Dump

```bash
sqlite3 logs/Accessibility/TCC.db "SELECT service, client, auth_value FROM access ORDER BY service"
```

### Find Sensitive Permissions

```bash
sqlite3 logs/Accessibility/TCC.db "
SELECT service, client FROM access
WHERE auth_value = 2
AND service IN (
    'kTCCServiceCamera',
    'kTCCServiceMicrophone',
    'kTCCServicePhotos',
    'kTCCServiceLocation',
    'kTCCServiceAddressBook'
)
"
```

### Compare Between Archives

```bash
# Extract permission sets
sqlite3 archive1/logs/Accessibility/TCC.db \
    "SELECT service||'|'||client||'|'||auth_value FROM access" | sort > /tmp/tcc1.txt
sqlite3 archive2/logs/Accessibility/TCC.db \
    "SELECT service||'|'||client||'|'||auth_value FROM access" | sort > /tmp/tcc2.txt

# Find changes
diff /tmp/tcc1.txt /tmp/tcc2.txt
```

## Forensic Analysis

### Permission Timeline

Track when permissions were granted/revoked:

```sql
SELECT service, client, auth_value,
       datetime(last_modified, 'unixepoch') as when_changed
FROM access
WHERE client = 'com.example.app'
ORDER BY last_modified;
```

### Detect Suspicious Patterns

1. **Apps with many permissions**: May indicate overprivileged apps
2. **Recent bulk permission changes**: May indicate compromise
3. **Location + Contacts + Photos**: High-risk combination
4. **Camera/Mic without UI presence**: Background surveillance risk

### Cross-Reference with Logs

```bash
# Find TCC-related log entries
log show --archive system_logs.logarchive \
    --predicate 'subsystem == "com.apple.TCC"'
```

## Privacy Implications

- TCC.db reveals app behavior patterns
- Permission timestamps show user interactions
- Denied permissions show privacy-conscious choices
- MDM policies show corporate control

## Related Files

| File | Description |
|------|-------------|
| `security-sysdiagnose.txt` | Security overview |
| Unified logs (`com.apple.TCC`) | TCC daemon activity |
| `tccd` process in ps.txt | TCC daemon state |

## See Also

- [keychain.md](keychain.md) - Keychain analysis
- [biome.md](biome.md) - Behavioral data
- [../analysis/privacy-audit.md](../analysis/privacy-audit.md) - Privacy analysis workflow
