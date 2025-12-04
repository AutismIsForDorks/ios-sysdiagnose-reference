# PowerLog Database (PLSQL)

The PowerLog database (`logs/powerlogs/*.PLSQL`) is a comprehensive SQLite database containing power, battery, and system telemetry data. It's one of the richest forensic artifacts in sysdiagnose.

## Location

```
logs/powerlogs/powerlog_YYYY-MM-DD_HH-MM_DEVICEID.PLSQL
```

## Database Overview

PowerLog contains 300+ tables organized by agent/subsystem:

```bash
sqlite3 powerlog.PLSQL ".tables" | wc -w
# Typically 300-400 tables
```

## Table Naming Convention

```
PL{Agent}_{EventType}_{DataType}
```

- **Agent**: System component (Battery, Application, Location, etc.)
- **EventType**: EventForward, EventBackward, EventInterval, EventPoint, Aggregate
- **DataType**: Specific data category

### Event Types

| Type | Description |
|------|-------------|
| `EventForward` | State changes (start events) |
| `EventBackward` | State changes (end events) |
| `EventInterval` | Duration-based measurements |
| `EventPoint` | Point-in-time events |
| `Aggregate` | Aggregated statistics |
| `EventNone` | Configuration/static data |

## Key Tables by Category

### Battery & Charging

```sql
-- Battery level over time
SELECT timestamp, Level, IsCharging, ExternalConnected
FROM PLBatteryAgent_EventBackward_Battery;

-- Charging sessions
SELECT * FROM PLBatteryAgent_EventInterval_Charging;

-- Battery health
SELECT * FROM PLBatteryAgent_EventBackward_TrustedBatteryHealth;

-- Smart charging
SELECT * FROM PLBatteryAgent_EventForward_SmartCharging;
```

### Application Usage

```sql
-- App runtime
SELECT * FROM PLAppTimeService_Aggregate_AppRunTime;

-- App usage events
SELECT * FROM PLAppTimeService_Aggregate_AppUsageEvents;

-- Foreground apps
SELECT * FROM PLApplicationAgent_EventForward_Application;

-- Widget updates
SELECT * FROM PLApplicationAgent_EventPoint_WidgetUpdates;
```

### Location

```sql
-- GPS power consumption
SELECT * FROM PLLocationAgent_EventBackward_GPSPower;

-- Location client activity
SELECT * FROM PLLocationAgent_EventForward_ClientStatus;

-- Significant locations
SELECT * FROM PLLocationAgent_EventBackward_LocationPower;
```

### Display & Touch

```sql
-- Display state
SELECT * FROM PLDisplayAgent_EventForward_Display;

-- Brightness changes
SELECT * FROM PLDisplayAgent_EventPoint_UserBrightness;

-- Touch events
SELECT * FROM PLDisplayAgent_EventBackward_Touch;

-- Ambient light
SELECT * FROM PLDisplayAgent_EventBackward_AmbientLight;
```

### Network

```sql
-- WiFi power
SELECT * FROM PLWifiAgent_EventBackward_APPower;

-- Cellular power
SELECT * FROM PLBBAgent_EventBackward_BBMavPeriodicMetrics;

-- Bluetooth
SELECT * FROM PLBluetoothAgent_EventInterval_ConnectedDevices;
```

### AI/ML (iOS 18+)

```sql
-- Generative AI opt-in
SELECT * FROM GenerativeFunctionMetrics_OptIn_1_2;

-- Summarization usage
SELECT * FROM GenerativeFunctionMetrics_Summarization_1_2;

-- Apple Diffusion (Genmoji)
SELECT * FROM GenerativeFunctionMetrics_appleDiffusion_1_2;

-- Model loading
SELECT * FROM GenerativeFunctionMetrics_assetLoad_1_2;

-- ML inference
SELECT * FROM ANE_modelInference_1_2;

-- Neural Engine status
SELECT * FROM ANE_ANEStatus_1_2;
```

### System State

```sql
-- Memory pressure
SELECT * FROM OSIMemory_MemoryState_2_2;

-- CPU cluster usage
SELECT * FROM PLCLPCAgent_EventInterval_CPUClusterAccumulators;

-- Thermal state
SELECT * FROM PLIOReportAgent_EventBackward_EnergyModel;
```

## Common Queries

### Battery Drain Analysis

```sql
-- Battery level timeline
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    Level,
    IsCharging,
    ExternalConnected
FROM PLBatteryAgent_EventBackward_Battery
ORDER BY timestamp;
```

### App Screen Time

```sql
-- Top apps by usage
SELECT
    BundleID,
    SUM(ScreenOnTime) as total_screen_time
FROM PLAppTimeService_Aggregate_AppRunTime
GROUP BY BundleID
ORDER BY total_screen_time DESC
LIMIT 20;
```

### Location Client Activity

```sql
-- Apps using location
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    Client,
    Type,
    Accuracy
FROM PLLocationAgent_EventForward_ClientStatus
WHERE Type != 0
ORDER BY timestamp DESC;
```

### Display Usage

```sql
-- Screen on/off timeline
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    State
FROM PLDisplayAgent_EventForward_Display
ORDER BY timestamp;
```

### Neural Engine Usage

```sql
-- ML model inference
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    modelName,
    inferenceTimeMs,
    inputSizeBytes
FROM ANE_modelInference_1_2
ORDER BY timestamp DESC
LIMIT 50;
```

## Timestamp Handling

PowerLog uses various timestamp formats:

```sql
-- Unix epoch (most common)
datetime(timestamp, 'unixepoch', 'localtime')

-- Cocoa epoch (seconds since 2001-01-01)
datetime(timestamp + 978307200, 'unixepoch', 'localtime')

-- Nanoseconds
datetime(timestamp/1000000000, 'unixepoch', 'localtime')
```

## Schema Discovery

### List All Tables

```bash
sqlite3 powerlog.PLSQL ".tables"
```

### Get Table Schema

```bash
sqlite3 powerlog.PLSQL ".schema PLBatteryAgent_EventBackward_Battery"
```

### Sample Table Data

```bash
sqlite3 powerlog.PLSQL "SELECT * FROM PLBatteryAgent_EventBackward_Battery LIMIT 5"
```

### Count Records

```bash
sqlite3 powerlog.PLSQL "
SELECT name, (SELECT COUNT(*) FROM (SELECT 1 FROM \"\" || name || \"\" LIMIT 1000))
FROM sqlite_master
WHERE type='table'
ORDER BY name
" 2>/dev/null | head -50
```

## Forensic Analysis

### Activity Timeline

```bash
# Combine battery, display, and app data
sqlite3 powerlog.PLSQL "
SELECT 'battery' as type, timestamp, Level as value FROM PLBatteryAgent_EventBackward_Battery
UNION ALL
SELECT 'display', timestamp, State FROM PLDisplayAgent_EventForward_Display
ORDER BY timestamp
" | head -100
```

### Power Drain Correlation

```bash
# Find battery drain periods
sqlite3 powerlog.PLSQL "
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    Level,
    Level - LAG(Level) OVER (ORDER BY timestamp) as drain
FROM PLBatteryAgent_EventBackward_Battery
WHERE drain IS NOT NULL AND drain < -5
ORDER BY timestamp
"
```

### AI Feature Usage

```bash
# Apple Intelligence activity
sqlite3 powerlog.PLSQL "
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    *
FROM GenerativeFunctionMetrics_Summarization_1_2
ORDER BY timestamp DESC
LIMIT 20
"
```

## Delta Analysis

Compare PowerLog data across archives:

```bash
for metric in "PLBatteryAgent_EventBackward_Battery" "GenerativeFunctionMetrics_OptIn_1_2"; do
    echo "=== $metric ==="
    for archive in baseline enabled disabled; do
        count=$(sqlite3 "$archive/logs/powerlogs/"*.PLSQL "SELECT COUNT(*) FROM $metric" 2>/dev/null || echo 0)
        echo "$archive: $count records"
    done
done
```

## Metadata Tables

### PLCoreStorage_Metadata

Contains database metadata:

```sql
SELECT * FROM PLCoreStorage_Metadata;
SELECT * FROM PLCoreStorage_MetadataVersion;
```

### Schema Versions

```sql
SELECT * FROM PLCoreStorage_schemaVersions;
```

## Size Management

PowerLog can grow large (10-40MB). Apple uses:
- Rolling retention (older data deleted)
- Aggregation (detailed → summary)
- Compression for older entries

## Related Files

| File | Description |
|------|-------------|
| `logs/powerlogs/` | PowerLog directory |
| `logs/BatteryBDC/` | Battery diagnostics |
| `logs/BatteryHealth/` | Battery health data |
| `logs/BatteryUIPlist/` | Battery UI state |

## See Also

- [../databases/schemas/powerlog.md](../databases/schemas/powerlog.md) - Full schema reference
- [../subsystems/intelligence.md](../subsystems/intelligence.md) - AI metrics tables
- [../analysis/delta-comparison.md](../analysis/delta-comparison.md) - Archive comparison
