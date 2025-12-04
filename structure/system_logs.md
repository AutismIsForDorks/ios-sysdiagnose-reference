# system_logs.logarchive Structure

The `system_logs.logarchive` directory contains the iOS Unified Logging System data in Apple's proprietary tracev3 format. This is the richest source of diagnostic data in a sysdiagnose.

## Directory Layout

```
system_logs.logarchive/
├── Info.plist              # Archive metadata
├── logdata.LiveData.tracev3  # Live/current log data
├── Persist/                # Persistent log chunks
│   ├── 0000000000000001.tracev3
│   ├── 0000000000000002.tracev3
│   └── ...
├── Special/                # Special category logs (longer TTL)
│   └── *.tracev3
├── Signpost/               # Signpost/performance data
│   └── *.tracev3
├── Extra/                  # Extra metadata
├── timesync/               # Time synchronization data
├── dsc/                    # Shared cache info
└── 00/ - FF/               # Hex-named directories (UUID-keyed chunks)
    └── *.tracev3
```

## Info.plist Metadata

```xml
{
  "ArchiveIdentifier": "267BEFE8-2AB5-4593-94C6-EA7FDEA0B53E",
  "OSArchiveVersion": 5,
  "OSLoggingSupportVersion": 1815.4,
  "EndTimeRef": {
    "ContinuousTime": 23596357608,
    "WallTime": 1763858319000000000,
    "UUID": "2D00B76C-D1DC-42B9-BF8B-6E5D920A8224"
  },
  "PersistSizeLimit": 146800640,    # ~140 MB
  "SignpostSizeLimit": 20971520,    # ~20 MB
  "SpecialSizeLimit": 73400320,     # ~70 MB
  "HighVolumeSizeLimit": 10485760   # ~10 MB
}
```

### Key Fields

| Field | Description |
|-------|-------------|
| `ArchiveIdentifier` | Unique ID for this archive |
| `OSArchiveVersion` | Log format version (5 = iOS 17+) |
| `EndTimeRef.WallTime` | Capture timestamp (nanoseconds since 1970) |
| `EndTimeRef.UUID` | Boot session UUID |
| `PersistSizeLimit` | Max size for persistent logs |

## TraceV3 File Categories

### Persist/ Directory
- **Purpose**: Standard persistent logs
- **Retention**: Days to weeks
- **Content**: General system logging
- **Typical count**: 5-20 files
- **Size per file**: ~10 MB

### Special/ Directory
- **Purpose**: High-priority logs with longer retention
- **Retention**: Weeks to months
- **Content**: Security events, crash-related logs
- **TTL**: Configurable per-subsystem

### Signpost/ Directory
- **Purpose**: Performance instrumentation
- **Content**: os_signpost intervals, performance markers
- **Use case**: App launch times, animation performance

### Hex Directories (00-FF)
- **Purpose**: UUID-keyed log chunks
- **Organization**: First byte of sender UUID
- **Content**: Per-process/subsystem logs

## Reading Log Archives

### Using `log show` (Recommended)

```bash
# Basic usage
log show --archive /path/to/system_logs.logarchive

# JSON output for parsing
log show --archive /path/to/system_logs.logarchive --style json

# Filter by predicate
log show --archive /path/to/system_logs.logarchive \
    --predicate 'subsystem == "com.apple.springboard"'

# Filter by process
log show --archive /path/to/system_logs.logarchive \
    --predicate 'process == "SpringBoard"'

# Time range
log show --archive /path/to/system_logs.logarchive \
    --start "2025-11-22 19:00:00" \
    --end "2025-11-22 20:00:00"

# Count events matching predicate
log show --archive /path/to/system_logs.logarchive \
    --predicate 'eventMessage CONTAINS "error"' \
    --style json | grep -c '"timestamp"'
```

### Common Predicates

```bash
# By subsystem
--predicate 'subsystem == "com.apple.springboard"'

# By process name
--predicate 'process == "mediaanalysisd"'

# By message content (case-insensitive)
--predicate 'eventMessage CONTAINS[c] "apple intelligence"'

# By log level
--predicate 'messageType == error'
--predicate 'messageType == fault'

# Combined filters
--predicate 'subsystem == "com.apple.siri" AND messageType == error'

# Exclusions
--predicate 'NOT subsystem == "com.apple.network"'
```

### JSON Output Fields

```json
{
  "timestamp": "2025-11-22 19:30:00.123456-0500",
  "processID": 1234,
  "processImagePath": "/usr/libexec/mediaanalysisd",
  "subsystem": "com.apple.photoanalysis",
  "category": "analysis",
  "eventMessage": "Starting photo analysis batch",
  "messageType": "Default",
  "senderImagePath": "/System/Library/PrivateFrameworks/MediaAnalysis.framework/MediaAnalysis",
  "threadID": 5678
}
```

## Log Categories

### By Size (Typical)

| Category | Typical Events | Storage |
|----------|----------------|---------|
| Network | 100K+ | High Volume |
| Security | 10K+ | Special (long TTL) |
| UI/SpringBoard | 50K+ | Persist |
| Background Tasks | 20K+ | Persist |
| Privacy/TCC | 5K+ | Special |

### By Forensic Value

| Priority | Subsystems |
|----------|------------|
| **Critical** | com.apple.tcc, com.apple.security, com.apple.locationd |
| **High** | com.apple.springboard, com.apple.siri, com.apple.photos |
| **Medium** | com.apple.network, com.apple.bluetooth |
| **Low** | com.apple.runningboard, com.apple.xpc |

## Privacy Redaction

iOS logs redact sensitive data by default:

```
eventMessage: "User location: <private>"
eventMessage: "Connected to network: <private>"
```

### Unredacted Access
- Requires device with logging profile installed
- Or MDM-managed device with debug entitlements
- Production sysdiagnoses always have redaction

## Time Synchronization

The `timesync/` directory contains:
- Boot session UUIDs
- Continuous time ↔ wall clock mapping
- Timezone information

This allows correlating logs across reboots.

## Performance Tips

### Large Archives
```bash
# Stream processing (don't load all into memory)
log show --archive /path/to/logarchive --style json 2>/dev/null | \
    grep -o '"subsystem" *: *"[^"]*"' | sort | uniq -c | sort -rn

# Limit output
log show --archive /path/to/logarchive --predicate '...' | head -1000
```

### Parallel Processing
```bash
# Process multiple predicates
for subsys in "com.apple.siri" "com.apple.springboard" "com.apple.photos"; do
    log show --archive /path/to/logarchive \
        --predicate "subsystem == \"$subsys\"" \
        --style json > "${subsys}.json" &
done
wait
```

## Common Issues

### Empty Output
- Archive may be from different OS version
- Predicate syntax error (check quoting)
- Time range outside archive bounds

### Slow Performance
- Large archives (>100MB) take time
- Use predicates to filter early
- Consider extracting specific subsystems

### Missing Logs
- High-volume logs may be overwritten
- Check Special/ for longer-retention events
- Some subsystems require debug profiles

## See Also

- [../formats/tracev3.md](../formats/tracev3.md) - TraceV3 binary format
- [../analysis/common-queries.md](../analysis/common-queries.md) - Useful log queries
- [../subsystems/index.md](../subsystems/index.md) - Subsystem reference
