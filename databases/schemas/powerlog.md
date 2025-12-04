# PowerLog Database Schema Reference

Complete schema reference for the PowerLog database (`.PLSQL`).

**Location**: `logs/powerlogs/powerlog_*.PLSQL`

---

## Overview

PowerLog contains 600+ tables (iOS 18.1). Tables are organized by:

- **Agent**: System component collecting data
- **Event Type**: Forward, Backward, Interval, Point, Aggregate
- **Data Type**: Specific measurement category

### Naming Convention

```
PL{Agent}_{EventType}_{DataType}
```

or

```
{Category}_{Subcategory}_{Version}
```

---

## Key Table Schemas

### Battery

#### PLBatteryAgent_EventBackward_Battery

Primary battery state table.

```sql
CREATE TABLE PLBatteryAgent_EventBackward_Battery (
    ID                      INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp               REAL,
    Level                   REAL,       -- Battery percentage (0-1)
    RawLevel                REAL,
    AtCriticalLevel         INTEGER,
    Voltage                 INTEGER,
    InstantAmperage         INTEGER,
    CurrentCapacity         INTEGER,
    MaxCapacity             INTEGER,
    DesignCapacity          INTEGER,
    CycleCount              INTEGER,
    ChargeStatus            TEXT,
    IsCharging              INTEGER,    -- 1 = charging
    FullyCharged            INTEGER,
    Amperage                INTEGER,
    Temperature             REAL,       -- Celsius
    ExternalConnected       INTEGER,    -- 1 = plugged in
    -- ... 70+ additional columns
);
```

**Key columns for forensics**:
- `timestamp`, `Level`, `IsCharging`, `ExternalConnected`

---

### Application Usage

#### PLAppTimeService_Aggregate_AppRunTime

App screen time aggregates.

```sql
CREATE TABLE PLAppTimeService_Aggregate_AppRunTime (
    ID                              INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp                       REAL,
    timeInterval                    REAL,
    BackgroundAudioNowPlayingPluggedInTime REAL,
    BackgroundAudioNowPlayingTime   REAL,
    BackgroundAudioPlayingTime      REAL,
    BackgroundLocationTime          REAL,
    BackgroundTime                  REAL,
    BundleID                        TEXT,       -- App identifier
    ScreenOnPluggedInTime           REAL,
    ScreenOnTime                    REAL        -- Foreground time
);

-- Indexes for efficient querying
CREATE INDEX Index_...timestamp ON ... (timestamp);
CREATE INDEX Index_...BundleID ON ... (BundleID);
```

---

### Location

#### PLLocationAgent_EventForward_ClientStatus

Location access by apps.

```sql
CREATE TABLE PLLocationAgent_EventForward_ClientStatus (
    ID                      INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp               REAL,
    timestampLogged         REAL,
    BundleId                TEXT,
    Client                  TEXT,
    Executable              TEXT,
    InUseLevel              INTEGER,
    LocationDesiredAccuracy REAL,
    LocationDistanceFilter  REAL,
    Type                    TEXT,
    timestampEnd            REAL
);
```

---

### Display

#### PLDisplayAgent_EventForward_Display

Screen state changes.

```sql
CREATE TABLE PLDisplayAgent_EventForward_Display (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    State       INTEGER     -- 0=off, 1=on
);
```

---

## AI/ML Tables (iOS 18+)

### GenerativeFunctionMetrics_OptIn_1_2

Apple Intelligence opt-in status.

```sql
CREATE TABLE GenerativeFunctionMetrics_OptIn_1_2 (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    Enabled     INTEGER     -- 0=disabled, 1=enabled
);
```

### GenerativeFunctionMetrics_Summarization_1_2

Summarization usage.

```sql
CREATE TABLE GenerativeFunctionMetrics_Summarization_1_2 (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    -- Additional metrics columns
);
```

### GenerativeFunctionMetrics_appleDiffusion_1_2

Image generation (Genmoji).

```sql
CREATE TABLE GenerativeFunctionMetrics_appleDiffusion_1_2 (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    -- Diffusion model metrics
);
```

### GenerativeFunctionMetrics_assetLoad_1_2

ML model asset loading.

```sql
CREATE TABLE GenerativeFunctionMetrics_assetLoad_1_2 (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    -- Asset loading metrics
);
```

### GenerativeFunctionMetrics_SmartReplySession_1_2

Smart reply sessions.

```sql
CREATE TABLE GenerativeFunctionMetrics_SmartReplySession_1_2 (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    -- Session metrics
);
```

### ANE_modelInference_1_2

Neural Engine inference.

```sql
CREATE TABLE ANE_modelInference_1_2 (
    ID              INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp       REAL,
    csIdentity      TEXT,       -- Code signing identity
    failureReason   INTEGER,
    modelURL        TEXT,       -- Model path/identifier
    programHandle   INTEGER
);
```

### ANE_ANEStatus_1_2

Neural Engine status.

```sql
CREATE TABLE ANE_ANEStatus_1_2 (
    ID          INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp   REAL,
    -- ANE status columns
);
```

---

## All GenerativeFunctionMetrics Tables

| Table | Purpose |
|-------|---------|
| `GenerativeFunctionMetrics_OptIn_1_2` | AI opt-in status |
| `GenerativeFunctionMetrics_Summarization_1_2` | Text summarization |
| `GenerativeFunctionMetrics_appleDiffusion_1_2` | Image generation |
| `GenerativeFunctionMetrics_SmartReplySession_1_2` | Smart replies |
| `GenerativeFunctionMetrics_HandwritingInference_1_2` | Handwriting ML |
| `GenerativeFunctionMetrics_HandwritingModelLoad_1_2` | Handwriting model loads |
| `GenerativeFunctionMetrics_PhotosGenerativeEdit_1_2` | Photos AI edits |
| `GenerativeFunctionMetrics_assetLoad_1_2` | Asset loading |
| `GenerativeFunctionMetrics_mmExecuteRequest_1_2` | Multi-modal requests |
| `GenerativeFunctionMetrics_tgiExecuteRequest_1_2` | Text generation |
| `GenerativeFunctionMetrics_fileResidentInfo_1_2` | File residency |

---

## Table Categories

### PL*Agent* Tables (~200)

| Prefix | Agent |
|--------|-------|
| `PLBatteryAgent_*` | Battery |
| `PLApplicationAgent_*` | Applications |
| `PLLocationAgent_*` | Location |
| `PLDisplayAgent_*` | Display |
| `PLWifiAgent_*` | WiFi |
| `PLBluetoothAgent_*` | Bluetooth |
| `PLBBAgent_*` | Baseband/cellular |
| `PLCameraAgent_*` | Camera |
| `PLAudioAgent_*` | Audio |

### Metrics Tables (~100)

| Prefix | Category |
|--------|----------|
| `ANE_*` | Neural Engine |
| `GenerativeFunctionMetrics_*` | AI/ML |
| `AccessibilityMetrics_*` | Accessibility |
| `BatteryMetrics_*` | Battery health |
| `ConfigMetrics_*` | Configuration |

### System Tables

| Table | Purpose |
|-------|---------|
| `PLCoreStorage_Metadata` | Database metadata |
| `PLCoreStorage_schemaVersions` | Schema versions |

---

## Timestamp Handling

All timestamps are Unix epoch (seconds since 1970-01-01):

```sql
-- Convert to readable format
SELECT datetime(timestamp, 'unixepoch', 'localtime') FROM table;

-- Filter by date
SELECT * FROM table
WHERE timestamp > strftime('%s', '2025-11-22');
```

---

## Common Queries

### Battery Timeline

```sql
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    Level * 100 as percent,
    IsCharging,
    ExternalConnected
FROM PLBatteryAgent_EventBackward_Battery
ORDER BY timestamp DESC
LIMIT 50;
```

### Top Apps by Screen Time

```sql
SELECT
    BundleID,
    SUM(ScreenOnTime) / 3600.0 as hours
FROM PLAppTimeService_Aggregate_AppRunTime
GROUP BY BundleID
ORDER BY hours DESC
LIMIT 20;
```

### Location Clients

```sql
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    BundleId,
    Client,
    Type
FROM PLLocationAgent_EventForward_ClientStatus
ORDER BY timestamp DESC
LIMIT 50;
```

### AI Opt-In History

```sql
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    Enabled
FROM GenerativeFunctionMetrics_OptIn_1_2
ORDER BY timestamp;
```

### Neural Engine Usage

```sql
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    modelURL,
    csIdentity
FROM ANE_modelInference_1_2
ORDER BY timestamp DESC
LIMIT 50;
```

---

## Listing All Tables

```bash
sqlite3 powerlog.PLSQL "SELECT name FROM sqlite_master WHERE type='table' ORDER BY name"
```

## Getting Any Table Schema

```bash
sqlite3 powerlog.PLSQL ".schema TABLE_NAME"
```

---

## See Also

- [tcc.md](tcc.md) - TCC schema
- [../../power/powerlog.md](../../power/powerlog.md) - PowerLog analysis guide
- [../../subsystems/intelligence.md](../../subsystems/intelligence.md) - AI metrics
