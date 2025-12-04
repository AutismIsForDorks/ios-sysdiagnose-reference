# iOS 17 Sysdiagnose Notes

Version-specific information for iOS 17.x sysdiagnose archives.

## Build Reference

| Build | Version | Notes |
|-------|---------|-------|
| 21A326 | iOS 17.0 | Initial release |
| 21B74 | iOS 17.1 | |
| 21C62 | iOS 17.2 | |
| 21D61 | iOS 17.3 | |
| 21E236 | iOS 17.4 | EU App Store changes |
| 21F79 | iOS 17.5 | |
| 21G93 | iOS 17.6 | |

---

## iOS 17 Structure

### Archive Layout

Same top-level structure as iOS 18:
```
sysdiagnose_*/
├── system_logs.logarchive/
├── logs/
├── crashes_and_spins/
├── WiFi/
├── ps.txt
├── ...
```

### Log Archive Version

```
OSArchiveVersion: 5
OSLoggingSupportVersion: ~1600.x
```

---

## Differences from iOS 18

### No Apple Intelligence

iOS 17 does NOT include:
- `intelligenceplatformd`
- `modelcatalogd`
- `logs/GenerativeExperiences/`
- `GenerativeFunctionMetrics_*` tables
- Intelligence-related Trial namespaces

### Siri/ML Differences

iOS 17 has simpler ML stack:

| iOS 17 Process | iOS 18 Equivalent |
|----------------|-------------------|
| `siriinferenced` | Same (enhanced) |
| `assistantd` | Same |
| N/A | `intelligenceplatformd` |
| N/A | `modelcatalogd` |

### PowerLog Tables

iOS 17 PowerLog has fewer tables (~250 vs ~350):

Missing in iOS 17:
```
GenerativeFunctionMetrics_*
```

Present in both:
```
PLBatteryAgent_*
PLApplicationAgent_*
PLLocationAgent_*
ANE_modelInference_*
```

### Trial System

Simpler Trial namespace structure:
- Fewer namespaces
- No AI_SAFETY_ADAPTORS
- No INTELLIGENCE_* namespaces

---

## iOS 17 Specific Artifacts

### Journal App (iOS 17.2+)

New in iOS 17.2:
```
logs/Journal/
```

Journal suggestions powered by:
- `suggestionsd`
- Biome behavioral data

### Standby Mode

New display mode tracked in:
```bash
--predicate 'eventMessage CONTAINS "standby" OR eventMessage CONTAINS "StandBy"'
```

### Contact Poster

New feature, tracked via:
- `com.apple.contacts` subsystem
- Photos/media analysis

---

## iOS 17 Queries

### Siri Activity

```bash
log show --archive system_logs.logarchive \
    --predicate 'subsystem BEGINSWITH "com.apple.siri"' \
    --style json | grep -c '"timestamp"'
```

### ML Inference (Pre-Apple Intelligence)

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "siriinferenced" OR eventMessage CONTAINS "inference"' \
    --style json
```

### Photo Analysis

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "photoanalysisd" OR process == "mediaanalysisd"' \
    --style json | grep -c '"timestamp"'
```

---

## TCC.db Compatibility

iOS 17 TCC.db schema is compatible with iOS 18:

```sql
-- Same tables
access
policies
active_policy
access_overrides
expired
admin

-- Same access table columns
service, client, client_type, auth_value, auth_reason, ...
```

---

## Migration Notes

### Upgrading to iOS 18

When device upgrades from iOS 17 to iOS 18:
- PowerLog tables are migrated
- New AI tables created empty
- TCC permissions preserved
- Trial namespaces expanded

### Comparing iOS 17 vs 18 Archives

```bash
# Process count difference
echo "iOS 17: $(awk 'NR>1' ios17_archive/ps.txt | wc -l)"
echo "iOS 18: $(awk 'NR>1' ios18_archive/ps.txt | wc -l)"

# AI process presence
echo "iOS 17 AI processes:"
grep -c "intelligence" ios17_archive/ps.txt || echo "0"
echo "iOS 18 AI processes:"
grep -c "intelligence" ios18_archive/ps.txt
```

---

## See Also

- [ios18.md](ios18.md) - iOS 18 notes
- [../structure/overview.md](../structure/overview.md) - Archive structure
