# Timeline Construction

Building forensic timelines from sysdiagnose artifacts. This guide covers techniques for reconstructing user activity and system events.

## Timeline Sources

| Source | Data | Time Resolution |
|--------|------|-----------------|
| Unified logs | System events | Milliseconds |
| TCC.db | Permission changes | Seconds |
| PowerLog | Battery, usage | Seconds |
| WiFi CSVs | Network joins | Seconds |
| Crash reports | Crashes | Seconds |
| ps.txt | Process state | Snapshot only |

---

## Timestamp Formats

### Unix Epoch (Most Common)

Seconds since 1970-01-01 00:00:00 UTC.

```bash
# Convert in SQLite
datetime(timestamp, 'unixepoch', 'localtime')

# Convert in bash
date -r 1732000000
```

### Cocoa Epoch

Seconds since 2001-01-01 00:00:00 UTC.

```bash
# Convert in SQLite
datetime(timestamp + 978307200, 'unixepoch', 'localtime')
```

### ISO 8601

Human-readable format in logs.

```
2025-11-22 19:30:00.123456-0500
```

---

## Building a Basic Timeline

### Step 1: Define Time Range

```bash
# Get archive time from filename
# sysdiagnose_2025.11.22_19-36-23-0500_*
# Captured at: 2025-11-22 19:36:23 EST

# Typically want 24-48 hours before capture
START="2025-11-21 00:00:00"
END="2025-11-22 19:36:23"
```

### Step 2: Extract Events from Each Source

#### Unified Logs

```bash
log show --archive system_logs.logarchive \
    --start "$START" --end "$END" \
    --style json > /tmp/logs.json
```

#### TCC Permissions

```bash
sqlite3 logs/Accessibility/TCC.db "
SELECT
    'TCC' as source,
    datetime(last_modified, 'unixepoch', 'localtime') as time,
    service || ' -> ' || client as event,
    auth_value as detail
FROM access
ORDER BY last_modified
" > /tmp/tcc_events.csv
```

#### Battery Events

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT
    'Battery' as source,
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    'Level: ' || CAST(Level * 100 AS INTEGER) || '%' as event,
    IsCharging as detail
FROM PLBatteryAgent_EventBackward_Battery
ORDER BY timestamp
" > /tmp/battery_events.csv
```

#### WiFi Joins

```bash
# Extract CSV
tar -xzf WiFi/Entity_*_Join.csv.tgz -C /tmp/

# Format events
awk -F',' 'NR>1 {
    cmd = "date -r " $1 " \"+%Y-%m-%d %H:%M:%S\""
    cmd | getline time
    close(cmd)
    print "WiFi," time ",Join: " $2 ","
}' /tmp/Join.csv > /tmp/wifi_events.csv
```

### Step 3: Merge Timeline

```bash
# Combine all sources
cat /tmp/tcc_events.csv /tmp/battery_events.csv /tmp/wifi_events.csv | \
    sort -t',' -k2 > /tmp/timeline.csv
```

---

## Event-Specific Timelines

### Application Usage Timeline

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    BundleID,
    ScreenOnTime as foreground_seconds
FROM PLAppTimeService_Aggregate_AppRunTime
WHERE ScreenOnTime > 60
ORDER BY timestamp
"
```

### Location Timeline

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "locationd" AND eventMessage CONTAINS "location"' \
    --style json | \
    grep -o '"timestamp" *: *"[^"]*"' | \
    cut -d'"' -f4 | sort
```

### Screen State Timeline

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    CASE State WHEN 0 THEN 'Screen OFF' ELSE 'Screen ON' END as event
FROM PLDisplayAgent_EventForward_Display
ORDER BY timestamp
"
```

### Communication Timeline

```bash
log show --archive system_logs.logarchive \
    --predicate '
        process == "identityservicesd" OR
        process == "imagent" OR
        process == "callservicesd"
    ' \
    --style json | \
    grep -E '"timestamp"|"eventMessage"' | \
    head -200
```

---

## Correlation Techniques

### Cross-Reference Events

1. **Find interesting time**: Identify a key event
2. **Window query**: Get events ±5 minutes around it
3. **Multi-source**: Query all sources for that window

```bash
# Found crash at 19:29:43
# Query logs around that time
log show --archive system_logs.logarchive \
    --start "2025-11-22 19:25:00" \
    --end "2025-11-22 19:35:00" \
    --predicate 'messageType == error OR messageType == fault' \
    --style json
```

### Activity Chains

Track causation across events:

1. App launch → TCC prompt → Permission grant → Feature use
2. WiFi join → Location update → App activity
3. Push notification → App launch → Network activity

### Gap Analysis

Find periods of no activity:

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT
    datetime(timestamp, 'unixepoch', 'localtime') as time,
    CAST((timestamp - LAG(timestamp) OVER (ORDER BY timestamp)) / 60 AS INTEGER) as gap_minutes
FROM PLBatteryAgent_EventBackward_Battery
WHERE gap_minutes > 30
ORDER BY timestamp
"
```

---

## Timeline Scripts

### Master Timeline Generator

```bash
#!/bin/bash
# generate_timeline.sh

ARCHIVE="$1"
OUTPUT="timeline.csv"

echo "timestamp,source,event,detail" > "$OUTPUT"

# TCC events
sqlite3 "$ARCHIVE/logs/Accessibility/TCC.db" "
SELECT last_modified, 'TCC', service || ' for ' || client,
       CASE auth_value WHEN 2 THEN 'allowed' ELSE 'denied' END
FROM access" >> "$OUTPUT"

# Battery events
sqlite3 "$ARCHIVE/logs/powerlogs/"*.PLSQL "
SELECT timestamp, 'Battery',
       'Level: ' || CAST(Level * 100 AS INTEGER) || '%',
       CASE IsCharging WHEN 1 THEN 'charging' ELSE '' END
FROM PLBatteryAgent_EventBackward_Battery" >> "$OUTPUT"

# Sort by timestamp
sort -t',' -k1 -n "$OUTPUT" -o "$OUTPUT"

echo "Timeline written to $OUTPUT"
```

### Interactive Timeline Viewer

```python
#!/usr/bin/env python3
# view_timeline.py

import sqlite3
import sys
from datetime import datetime

def format_time(ts):
    return datetime.fromtimestamp(ts).strftime('%Y-%m-%d %H:%M:%S')

def main(archive_path):
    # TCC events
    tcc_db = f"{archive_path}/logs/Accessibility/TCC.db"
    conn = sqlite3.connect(tcc_db)
    cursor = conn.execute("""
        SELECT last_modified, service, client, auth_value
        FROM access ORDER BY last_modified
    """)

    for row in cursor:
        ts, service, client, auth = row
        status = "ALLOWED" if auth == 2 else "DENIED"
        print(f"{format_time(ts)} [TCC] {service} -> {client}: {status}")

    conn.close()

if __name__ == "__main__":
    main(sys.argv[1])
```

---

## Visualization

### ASCII Timeline

```bash
# Simple timeline bars
sqlite3 logs/powerlogs/*.PLSQL "
SELECT
    strftime('%H', datetime(timestamp, 'unixepoch', 'localtime')) as hour,
    COUNT(*) as events
FROM PLBatteryAgent_EventBackward_Battery
GROUP BY hour
" | while read line; do
    hour=$(echo "$line" | cut -d'|' -f1)
    count=$(echo "$line" | cut -d'|' -f2)
    bar=$(printf '%*s' "$((count/10))" | tr ' ' '#')
    echo "$hour: $bar ($count)"
done
```

### Export for Visualization Tools

```bash
# CSV for Excel/Sheets
sqlite3 -header -csv logs/powerlogs/*.PLSQL "
SELECT
    datetime(timestamp, 'unixepoch') as time,
    Level,
    IsCharging
FROM PLBatteryAgent_EventBackward_Battery
" > battery_timeline.csv

# JSON for web tools
sqlite3 -json logs/powerlogs/*.PLSQL "..." > timeline.json
```

---

## Common Patterns

### Device Usage Session

```
19:00 - Screen ON
19:00 - App launch (Messages)
19:01 - iMessage activity
19:05 - App switch (Safari)
19:10 - Location request
19:15 - Screen OFF
```

### Privacy Event Chain

```
19:30 - App launch (Camera app)
19:30 - TCC prompt (Camera)
19:30 - User grants permission
19:31 - Camera active
19:35 - Photo saved
```

### Network Transition

```
08:00 - WiFi disconnect (home)
08:05 - Cellular connect
08:30 - WiFi connect (office)
08:31 - Location update
```

---

## Tips

1. **Start with battery** - Battery events are reliable time anchors
2. **Use screen state** - Screen ON/OFF brackets user interaction
3. **Correlate crashes** - Crash timestamps often reveal issues
4. **Watch for gaps** - Missing data may indicate device off or issues
5. **Timezone awareness** - Convert everything to consistent timezone

---

## See Also

- [delta-comparison.md](delta-comparison.md) - Comparing archives
- [common-queries.md](common-queries.md) - Log queries
- [../power/powerlog.md](../power/powerlog.md) - PowerLog data
- [../privacy/tcc.md](../privacy/tcc.md) - TCC events
