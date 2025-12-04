# logs/GenerativeExperiences/ - Apple Intelligence

**iOS 18+ only**

Contains Apple Intelligence availability and configuration data.

## Directory Contents

| File | Description |
|------|-------------|
| `GEAvailability.log` | Apple Intelligence availability status |

---

## GEAvailability.log

This file contains Apple Intelligence eligibility and availability checks.

### Sample Content

```
Apple Intelligence availability check
Device eligible: true
Region supported: true
Language supported: true
Opt-in state: false
Feature availability:
  - Writing Tools: available
  - Summarization: available
  - Genmoji: available
  - Image Playground: available
```

### Key Fields

| Field | Description |
|-------|-------------|
| Device eligible | Hardware supports AI |
| Region supported | Geographic availability |
| Language supported | Language support |
| Opt-in state | User consent status |
| Feature availability | Per-feature status |

---

## Parsing

### Quick Check

```bash
cat logs/GenerativeExperiences/GEAvailability.log
```

### Extract Opt-In State

```bash
grep -i "opt" logs/GenerativeExperiences/GEAvailability.log
```

### Check Feature Status

```bash
grep -i "available\|enabled\|disabled" logs/GenerativeExperiences/GEAvailability.log
```

---

## Cross-Reference with Logs

### AI Availability Events

```bash
log show --archive system_logs.logarchive \
    --predicate 'eventMessage CONTAINS "GEAvailability" OR eventMessage CONTAINS "eligibility"' \
    --style json
```

### AI Opt-In Messages

```bash
log show --archive system_logs.logarchive \
    --predicate 'eventMessage CONTAINS[c] "apple intelligence" AND eventMessage CONTAINS "opt"' \
    --style json
```

---

## Forensic Value

**High** - This file directly shows:

1. Whether device supports Apple Intelligence
2. User opt-in consent state
3. Which AI features are available
4. Regional/language restrictions

### Privacy Analysis

If `Opt-in state: false` but AI activity detected in logs, this indicates potential privacy violation.

### Compare Across Archives

```bash
for archive in baseline enabled disabled; do
    echo "=== $archive ==="
    cat "$archive/logs/GenerativeExperiences/GEAvailability.log" 2>/dev/null || echo "Not present"
done
```

---

## Related Artifacts

| Location | Data |
|----------|------|
| Unified logs | AI subsystem events |
| PowerLog | `GenerativeFunctionMetrics_*` tables |
| Trial | AI namespace NCV values |
| ps.txt | AI processes running |

---

## See Also

- [index.md](index.md) - Logs directory index
- [trial.md](trial.md) - Trial namespaces
- [../../subsystems/intelligence.md](../../subsystems/intelligence.md) - AI subsystems
- [../../power/powerlog.md](../../power/powerlog.md) - AI metrics tables
