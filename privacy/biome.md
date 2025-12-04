# Biome - Behavioral Intelligence System

Biome (previously Duet Activity Scheduler / DAS) is Apple's behavioral data collection framework. It powers features like Siri Suggestions, app predictions, and Apple Intelligence personalization.

## Overview

Biome collects and stores:
- App usage patterns
- Location visits
- Communication patterns (contacts, not content)
- Device interactions
- User routines

## Key Processes

| Process | Description |
|---------|-------------|
| `biomed` | Main Biome daemon |
| `biomesyncd` | iCloud sync for Biome |
| `duetexpertd` | Prediction engine |
| `coreduetd` | Core Duet scheduler |
| `contextstored` | Context storage |
| `routined` | Routine learning |
| `parsecd` | Siri/Search personalization |

## Biome Data Streams

Biome organizes data into "streams" - categorized behavioral signals:

### Device Streams

| Stream | Contents |
|--------|----------|
| `device/isLocked` | Lock/unlock events |
| `device/isPluggedIn` | Charging state |
| `device/batteryLevel` | Battery percentage |
| `device/screenState` | Screen on/off |

### App Streams

| Stream | Contents |
|--------|----------|
| `app/usage` | App foreground time |
| `app/install` | App installations |
| `app/inFocus` | Currently focused app |
| `app/intents` | Siri Shortcuts usage |

### Location Streams

| Stream | Contents |
|--------|----------|
| `location/visit` | Significant locations |
| `location/routine` | Predicted locations |
| `location/workout` | Exercise locations |

### Social Streams

| Stream | Contents |
|--------|----------|
| `social/contact` | Contact interactions |
| `social/message` | Message metadata (not content) |
| `social/call` | Call metadata |

### System Streams

| Stream | Contents |
|--------|----------|
| `system/notification` | Notification delivery |
| `system/widget` | Widget interactions |
| `system/spotlight` | Search queries (local) |

## Log Analysis

### Find Biome Events

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "biomed"' \
    --style json | head -1000
```

### Count Biome Activity

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "biomed"' \
    --style json | grep -c '"timestamp"'
```

### Find Stream Writes

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "biomed" AND eventMessage CONTAINS "stream"' \
    --style json
```

## PowerLog Biome Tables

The PowerLog database contains Biome-related tables:

```sql
-- App usage aggregates
SELECT * FROM PLAppTimeService_Aggregate_AppRunTime;

-- Device state
SELECT * FROM PLDuetService_EventForward_BatterySaverMode;

-- Predictions
SELECT * FROM PLDuetService_EventForward_DASPrediction;
```

## Privacy Implications

### Data Sensitivity

| Level | Data Type |
|-------|-----------|
| **Critical** | Location visits, contact patterns |
| **High** | App usage, search queries |
| **Medium** | Device state, notifications |
| **Low** | System events |

### Retention

- Local data: Up to 90 days
- iCloud sync: Encrypted, tied to account
- Purged on: Device wipe, account sign-out

### Controls

- Settings > Privacy > Analytics & Improvements
- Settings > Siri & Search (per-app)
- Settings > Privacy > Apple Advertising

## Forensic Analysis

### Activity Patterns

Cross-reference biomed logs with:
- Location events (locationd)
- App launches (runningboardd)
- Screen state (SpringBoard)

### Timeline Construction

```bash
# Biome + Location + App launches
log show --archive system_logs.logarchive \
    --predicate '
        process == "biomed" OR
        process == "locationd" OR
        process == "runningboardd"
    ' \
    --style json | grep -E '"timestamp"|"eventMessage"'
```

### Prediction Analysis

Find what the system predicts about user behavior:

```bash
log show --archive system_logs.logarchive \
    --predicate 'eventMessage CONTAINS "prediction"' \
    --style json
```

## Apple Intelligence Integration

In iOS 18+, Biome feeds Apple Intelligence:

### Semantic Index

Biome data helps build the on-device semantic index:
- Contact relationships
- App context
- Location relevance

### Siri Suggestions

Biome powers:
- App suggestions on lock screen
- Search ranking
- Notification priority

### Writing Tools

Context from Biome may influence:
- Smart replies
- Text predictions
- Genmoji suggestions

## Delta Analysis

Compare Biome activity across archives:

```bash
for archive in baseline enabled disabled; do
    echo "=== $archive ==="
    log show --archive "$archive/system_logs.logarchive" \
        --predicate 'process == "biomed"' \
        --style json 2>/dev/null | grep -c '"timestamp"'
done
```

## Related Subsystems

| Subsystem | Purpose |
|-----------|---------|
| `com.apple.biome` | Core Biome |
| `com.apple.duet` | Duet scheduler |
| `com.apple.parsec` | Siri personalization |
| `com.apple.routined` | Routine learning |
| `com.apple.contextstore` | Context storage |

## Logs Directory

Check `logs/` for Biome-related data:

```bash
ls logs/ | grep -iE "biome|duet|context|routine|parsec"
```

## See Also

- [tcc.md](tcc.md) - Permission database
- [location.md](location.md) - Location data
- [../power/powerlog.md](../power/powerlog.md) - PowerLog tables
- [../subsystems/intelligence.md](../subsystems/intelligence.md) - Apple Intelligence
