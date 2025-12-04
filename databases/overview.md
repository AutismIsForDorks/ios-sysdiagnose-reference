# SQLite Databases in Sysdiagnose

Sysdiagnose archives contain several SQLite databases with valuable forensic data. This document provides an overview of key databases and their purposes.

## Database Locations

| Path | Database | Purpose |
|------|----------|---------|
| `logs/Accessibility/TCC.db` | TCC | Privacy permissions |
| `logs/powerlogs/*.PLSQL` | PowerLog | Power/system telemetry |

## Tools

### SQLite3 CLI

```bash
# Open database
sqlite3 /path/to/database.db

# List tables
.tables

# Show schema
.schema tablename

# Export to CSV
.headers on
.mode csv
.output data.csv
SELECT * FROM tablename;
.output stdout
```

### Quick Queries

```bash
# Single query
sqlite3 database.db "SELECT * FROM table LIMIT 10"

# With headers
sqlite3 -header -column database.db "SELECT * FROM table"

# JSON output
sqlite3 -json database.db "SELECT * FROM table"
```

---

## TCC.db

**Location**: `logs/Accessibility/TCC.db`
**Purpose**: Privacy permission storage

### Key Tables

| Table | Purpose |
|-------|---------|
| `access` | Permission grants/denials |
| `policies` | MDM policies |
| `active_policy` | Active policy links |
| `access_overrides` | System overrides |
| `expired` | Revoked permissions |

### Quick Queries

```bash
# All permissions
sqlite3 logs/Accessibility/TCC.db "SELECT service, client, auth_value FROM access"

# Allowed permissions
sqlite3 logs/Accessibility/TCC.db "SELECT service, client FROM access WHERE auth_value = 2"

# Recent changes
sqlite3 logs/Accessibility/TCC.db "
SELECT service, client, datetime(last_modified, 'unixepoch')
FROM access
ORDER BY last_modified DESC
LIMIT 20"
```

**Full documentation**: [../privacy/tcc.md](../privacy/tcc.md)

---

## PowerLog (PLSQL)

**Location**: `logs/powerlogs/powerlog_*.PLSQL`
**Purpose**: System telemetry and power data

### Categories

| Prefix | Category |
|--------|----------|
| `PL*Agent*` | Agent-collected data |
| `ANE_*` | Neural Engine metrics |
| `GenerativeFunctionMetrics_*` | AI/ML metrics |
| `OSIMemory_*` | Memory state |

### Table Count

```bash
sqlite3 powerlog.PLSQL ".tables" | wc -w
# Typically 300-400 tables
```

### Key Tables

```bash
# Battery
sqlite3 powerlog.PLSQL "SELECT * FROM PLBatteryAgent_EventBackward_Battery LIMIT 5"

# App usage
sqlite3 powerlog.PLSQL "SELECT * FROM PLAppTimeService_Aggregate_AppRunTime LIMIT 5"

# AI metrics
sqlite3 powerlog.PLSQL "SELECT * FROM GenerativeFunctionMetrics_OptIn_1_2"
```

**Full documentation**: [../power/powerlog.md](../power/powerlog.md)

---

## Database Schema Discovery

### List All Tables

```bash
sqlite3 database.db "SELECT name FROM sqlite_master WHERE type='table' ORDER BY name"
```

### Get Table Schema

```bash
sqlite3 database.db ".schema tablename"
```

### Get Column Info

```bash
sqlite3 database.db "PRAGMA table_info(tablename)"
```

### Count Records Per Table

```bash
sqlite3 database.db "
SELECT name,
       (SELECT COUNT(*) FROM \"\" || name || \"\") as count
FROM sqlite_master
WHERE type='table'
" 2>/dev/null | head -50
```

---

## Common Query Patterns

### Timestamp Conversion

```sql
-- Unix epoch
datetime(timestamp, 'unixepoch', 'localtime')

-- Cocoa epoch (seconds since 2001-01-01)
datetime(timestamp + 978307200, 'unixepoch', 'localtime')

-- Nanoseconds
datetime(timestamp/1000000000, 'unixepoch', 'localtime')
```

### Time Range Filter

```sql
WHERE timestamp > strftime('%s', 'now', '-7 days')
```

### Top N by Count

```sql
SELECT column, COUNT(*) as cnt
FROM table
GROUP BY column
ORDER BY cnt DESC
LIMIT 10
```

### Join Tables

```sql
SELECT a.*, b.extra_field
FROM table_a a
JOIN table_b b ON a.id = b.foreign_id
```

---

## Forensic Database Analysis

### Check Database Integrity

```bash
sqlite3 database.db "PRAGMA integrity_check"
```

### Get Database Stats

```bash
sqlite3 database.db "
SELECT
    (SELECT COUNT(*) FROM sqlite_master WHERE type='table') as tables,
    (SELECT page_count * page_size FROM pragma_page_count, pragma_page_size) as size_bytes
"
```

### Export for Analysis

```bash
# Dump entire database
sqlite3 database.db .dump > database.sql

# Export specific table to CSV
sqlite3 -header -csv database.db "SELECT * FROM tablename" > tablename.csv
```

### Compare Databases

```bash
# Schema diff
diff <(sqlite3 db1.db ".schema") <(sqlite3 db2.db ".schema")

# Record count comparison
for table in access policies expired; do
    echo "=== $table ==="
    echo -n "DB1: "; sqlite3 db1.db "SELECT COUNT(*) FROM $table"
    echo -n "DB2: "; sqlite3 db2.db "SELECT COUNT(*) FROM $table"
done
```

---

## Database Integrity Notes

### WAL Files

Some databases may have write-ahead log files (`.db-wal`, `.db-shm`). For complete data, ensure these are processed.

### Locked Databases

If a database was copied while in use, it may be inconsistent. Check with:

```bash
sqlite3 database.db "PRAGMA integrity_check"
```

### Schema Versions

Databases may have version metadata:

```sql
SELECT * FROM sqlite_master WHERE name LIKE '%version%' OR name LIKE '%meta%'
```

---

## See Also

- [schemas/tcc.md](schemas/tcc.md) - TCC schema details
- [schemas/powerlog.md](schemas/powerlog.md) - PowerLog schema details
- [../privacy/tcc.md](../privacy/tcc.md) - TCC analysis
- [../power/powerlog.md](../power/powerlog.md) - PowerLog analysis
