# logs/powerlogs/ - PowerLog Database

Contains the PowerLog SQLite database with comprehensive system telemetry.

## Directory Contents

| File Pattern | Description |
|--------------|-------------|
| `powerlog_YYYY-MM-DD_HH-MM_DEVICEID.PLSQL` | Main PowerLog database |

Example: `powerlog_2025-11-22_19-36_185426FB.PLSQL`

---

## Database Overview

PowerLog is a SQLite database containing 300-400 tables organized by system agent.

### Size

Typical size: 10-40 MB

### Table Count

```bash
sqlite3 logs/powerlogs/*.PLSQL ".tables" | wc -w
```

---

## Table Categories

### Battery & Power

| Table Pattern | Data |
|---------------|------|
| `PLBatteryAgent_*` | Battery level, charging, health |
| `PLCLPCAgent_*` | CPU power management |
| `PLIOReportAgent_*` | I/O power metrics |

### Application

| Table Pattern | Data |
|---------------|------|
| `PLApplicationAgent_*` | App lifecycle, memory |
| `PLAppTimeService_*` | Screen time, usage |

### Location

| Table Pattern | Data |
|---------------|------|
| `PLLocationAgent_*` | GPS, location clients |
| `CoreLocation_*` | Geofence triggers |

### Network

| Table Pattern | Data |
|---------------|------|
| `PLWifiAgent_*` | WiFi power |
| `PLBBAgent_*` | Cellular/baseband |
| `PLBluetoothAgent_*` | Bluetooth |

### Display

| Table Pattern | Data |
|---------------|------|
| `PLDisplayAgent_*` | Screen state, brightness |

### AI/ML (iOS 18+)

| Table Pattern | Data |
|---------------|------|
| `GenerativeFunctionMetrics_*` | Apple Intelligence |
| `ANE_*` | Neural Engine |

---

## Quick Queries

### List All Tables

```bash
sqlite3 logs/powerlogs/*.PLSQL ".tables"
```

### Battery Timeline

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT datetime(timestamp, 'unixepoch', 'localtime'), Level, IsCharging
FROM PLBatteryAgent_EventBackward_Battery
ORDER BY timestamp DESC
LIMIT 20"
```

### App Usage

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT BundleID, SUM(ScreenOnTime) as total
FROM PLAppTimeService_Aggregate_AppRunTime
GROUP BY BundleID
ORDER BY total DESC
LIMIT 20"
```

### AI Metrics

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT * FROM GenerativeFunctionMetrics_OptIn_1_2
ORDER BY timestamp DESC"
```

---

## Full Documentation

See: [../../power/powerlog.md](../../power/powerlog.md)

---

## See Also

- [index.md](index.md) - Logs directory index
- [../../power/powerlog.md](../../power/powerlog.md) - Full PowerLog guide
- [../../databases/schemas/powerlog.md](../../databases/schemas/powerlog.md) - Schema reference
