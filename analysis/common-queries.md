# Common Log Queries

A collection of useful `log show` predicates and analysis commands for iOS sysdiagnose forensics.

## Basic Syntax

```bash
log show --archive /path/to/system_logs.logarchive \
    --predicate 'PREDICATE' \
    --style json \
    --start "YYYY-MM-DD HH:MM:SS" \
    --end "YYYY-MM-DD HH:MM:SS"
```

---

## Predicate Reference

### Comparison Operators

| Operator | Usage |
|----------|-------|
| `==` | Equals |
| `!=` | Not equals |
| `<`, `>`, `<=`, `>=` | Numeric comparison |
| `CONTAINS` | String contains |
| `CONTAINS[c]` | Case-insensitive contains |
| `BEGINSWITH` | String starts with |
| `ENDSWITH` | String ends with |
| `MATCHES` | Regex match |
| `IN` | In set |

### Logical Operators

| Operator | Usage |
|----------|-------|
| `AND` | Both conditions |
| `OR` | Either condition |
| `NOT` | Negation |

### Common Fields

| Field | Description |
|-------|-------------|
| `process` | Process name |
| `processImagePath` | Full process path |
| `subsystem` | Logging subsystem |
| `category` | Log category |
| `eventMessage` | Log message |
| `messageType` | Log level (default, info, debug, error, fault) |

---

## Privacy & Security

### TCC Permission Events

```bash
--predicate 'subsystem == "com.apple.TCC"'
```

### Security Events

```bash
--predicate 'subsystem == "com.apple.security" OR subsystem == "com.apple.securityd"'
```

### Keychain Access

```bash
--predicate 'eventMessage CONTAINS "keychain" OR process == "securityd"'
```

### Location Access

```bash
--predicate 'process == "locationd" OR subsystem == "com.apple.locationd"'
```

### Privacy Accounting

```bash
--predicate 'subsystem == "com.apple.privacyaccounting"'
```

---

## Apple Intelligence

### AI Platform Events

```bash
--predicate 'subsystem BEGINSWITH "com.apple.intelligenceplatform"'
```

### AI Opt-In Status

```bash
--predicate 'eventMessage CONTAINS[c] "apple intelligence" AND eventMessage CONTAINS "opt"'
```

### Model Activity

```bash
--predicate 'process == "modelcatalogd" OR process == "modelmanagerd"'
```

### Inference Events

```bash
--predicate 'process == "siriinferenced" OR eventMessage CONTAINS "inference"'
```

### Summarization

```bash
--predicate 'eventMessage CONTAINS "summar" OR subsystem CONTAINS "summarization"'
```

---

## Siri & Search

### All Siri Events

```bash
--predicate 'subsystem BEGINSWITH "com.apple.siri"'
```

### Siri Requests

```bash
--predicate 'process == "assistantd" AND eventMessage CONTAINS "request"'
```

### Spotlight Indexing

```bash
--predicate 'process == "corespotlightd"'
```

### Search Queries

```bash
--predicate 'eventMessage CONTAINS "search" OR eventMessage CONTAINS "query"'
```

---

## Application Lifecycle

### App Launches

```bash
--predicate 'process == "runningboardd" AND eventMessage CONTAINS "launch"'
```

### App Terminations

```bash
--predicate 'process == "runningboardd" AND eventMessage CONTAINS "terminate"'
```

### Background Tasks

```bash
--predicate 'eventMessage CONTAINS "background" OR subsystem == "com.apple.runningboard"'
```

### Jetsam Events

```bash
--predicate 'eventMessage CONTAINS "jetsam" OR eventMessage CONTAINS "memory pressure"'
```

---

## Network

### WiFi Events

```bash
--predicate 'subsystem == "com.apple.wifi" OR process == "wifid"'
```

### WiFi Joins

```bash
--predicate 'process == "wifid" AND eventMessage CONTAINS "join"'
```

### Bluetooth

```bash
--predicate 'subsystem == "com.apple.bluetooth" OR process == "bluetoothd"'
```

### Push Notifications

```bash
--predicate 'process == "apsd" OR subsystem == "com.apple.apsd"'
```

### Network Changes

```bash
--predicate 'process == "configd" AND eventMessage CONTAINS "network"'
```

---

## Errors & Faults

### All Errors

```bash
--predicate 'messageType == error'
```

### All Faults

```bash
--predicate 'messageType == fault'
```

### Errors by Process

```bash
--predicate 'messageType == error AND process == "PROCESSNAME"'
```

### Crash-Related

```bash
--predicate 'eventMessage CONTAINS "crash" OR eventMessage CONTAINS "exception"'
```

---

## System Events

### Boot Events

```bash
--predicate 'eventMessage CONTAINS "boot" OR eventMessage CONTAINS "startup"'
```

### Sleep/Wake

```bash
--predicate 'eventMessage CONTAINS "sleep" OR eventMessage CONTAINS "wake"'
```

### Display State

```bash
--predicate 'process == "SpringBoard" AND eventMessage CONTAINS "display"'
```

### Lock/Unlock

```bash
--predicate 'eventMessage CONTAINS "lock" OR eventMessage CONTAINS "unlock"'
```

---

## Trial/Experiments

### Trial Events

```bash
--predicate 'subsystem == "com.apple.trial"'
```

### Experiment Enrollment

```bash
--predicate 'eventMessage CONTAINS "experiment" OR eventMessage CONTAINS "rollout"'
```

### Feature Flags

```bash
--predicate 'eventMessage CONTAINS "feature" AND eventMessage CONTAINS "flag"'
```

---

## Counting Patterns

### Count Events

```bash
log show --archive /path/to/logarchive \
    --predicate 'PREDICATE' \
    --style json 2>/dev/null | grep -c '"timestamp"'
```

### Count by Subsystem

```bash
log show --archive /path/to/logarchive --style json 2>/dev/null | \
    grep -o '"subsystem" *: *"[^"]*"' | sort | uniq -c | sort -rn | head -20
```

### Count by Process

```bash
log show --archive /path/to/logarchive --style json 2>/dev/null | \
    grep -o '"process" *: *"[^"]*"' | sort | uniq -c | sort -rn | head -20
```

---

## Time-Based Queries

### Last Hour

```bash
--start "$(date -v-1H '+%Y-%m-%d %H:%M:%S')"
```

### Specific Time Range

```bash
--start "2025-11-22 19:00:00" --end "2025-11-22 20:00:00"
```

### Events Around Crash

```bash
# Find crash time from .ips file, then:
--start "2025-11-22 19:28:00" --end "2025-11-22 19:30:00"
```

---

## Output Processing

### Extract Timestamps

```bash
log show ... --style json | grep -o '"timestamp" *: *"[^"]*"' | cut -d'"' -f4
```

### Extract Messages

```bash
log show ... --style json | grep -o '"eventMessage" *: *"[^"]*"' | cut -d'"' -f4
```

### JSON to CSV

```bash
log show ... --style json | python3 -c "
import json, sys, csv
writer = csv.writer(sys.stdout)
writer.writerow(['timestamp', 'process', 'message'])
for line in sys.stdin:
    try:
        obj = json.loads(line)
        writer.writerow([obj.get('timestamp',''), obj.get('processImagePath',''), obj.get('eventMessage','')])
    except: pass
"
```

### Unique Messages

```bash
log show ... --style json | grep -o '"eventMessage" *: *"[^"]*"' | sort | uniq -c | sort -rn
```

---

## Complex Queries

### AI Activity with Errors

```bash
--predicate '
    (process == "intelligenceplatformd" OR process == "modelcatalogd") AND
    messageType == error
'
```

### Privacy Access During Background

```bash
--predicate '
    (process == "locationd" OR process == "biomed") AND
    eventMessage CONTAINS "background"
'
```

### Excluding Noise

```bash
--predicate '
    subsystem == "com.apple.springboard" AND
    NOT eventMessage CONTAINS "animation" AND
    NOT eventMessage CONTAINS "frame"
'
```

---

## Performance Tips

### Limit Output

```bash
log show ... | head -1000
```

### Parallel Subsystem Extraction

```bash
for subsys in "com.apple.siri" "com.apple.security" "com.apple.TCC"; do
    log show --archive /path/to/logarchive \
        --predicate "subsystem == \"$subsys\"" \
        --style json > "/tmp/${subsys}.json" &
done
wait
```

### Stream Processing

```bash
log show ... --style json | while read line; do
    echo "$line" | grep -q "error" && echo "$line"
done
```

---

## See Also

- [delta-comparison.md](delta-comparison.md) - Comparing archives
- [timeline-construction.md](timeline-construction.md) - Building timelines
- [../structure/system_logs.md](../structure/system_logs.md) - Log archive structure
