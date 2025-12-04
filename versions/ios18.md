# iOS 18 / 26 Sysdiagnose Notes

Version-specific information for iOS 18.x (internal version 26.x) sysdiagnose archives.

**Note**: iOS uses dual versioning - marketing version (18.x) and internal version (26.x). Build 23B85 = iOS 18.1 (marketing) = iOS 26.1 (internal). Logs show the internal version.

## Tested Builds

| Build | Marketing | Internal | Device | Notes |
|-------|-----------|----------|--------|-------|
| 23B85 | iOS 18.1 | 26.1 | iPhone 16 Pro | Primary reference |

---

## New in iOS 18

### Apple Intelligence

iOS 18 introduces Apple Intelligence, adding significant new artifacts:

#### New Processes

| Process | Purpose |
|---------|---------|
| `intelligenceplatformd` | AI orchestration daemon |
| `IntelligencePlatformComputeService` | ML compute XPC |
| `modelcatalogd` | Model catalog management |
| `modelmanagerd` | Model lifecycle |

#### New Log Subsystems

```
com.apple.intelligenceplatform
com.apple.intelligenceplatform.compute
com.apple.intelligenceplatform.core
com.apple.generativeexperiences
com.apple.summarization
```

#### New Logs Directory

```
logs/GenerativeExperiences/
└── GEAvailability.log    # Apple Intelligence availability
```

#### New PowerLog Tables

```sql
GenerativeFunctionMetrics_OptIn_1_2
GenerativeFunctionMetrics_Summarization_1_2
GenerativeFunctionMetrics_appleDiffusion_1_2
GenerativeFunctionMetrics_SmartReplySession_1_2
GenerativeFunctionMetrics_assetLoad_1_2
GenerativeFunctionMetrics_HandwritingInference_1_2
GenerativeFunctionMetrics_HandwritingModelLoad_1_2
GenerativeFunctionMetrics_PhotosGenerativeEdit_1_2
GenerativeFunctionMetrics_mmExecuteRequest_1_2
GenerativeFunctionMetrics_tgiExecuteRequest_1_2
GenerativeFunctionMetrics_fileResidentInfo_1_2
```

### Trial System Updates

Enhanced Trial namespace structure for AI features:

| Namespace | Purpose |
|-----------|---------|
| `AI_SAFETY_ADAPTORS` | Safety system config |
| `DEVICE_EXPERT_ADAPTORS` | Device adaptations |
| `INTELLIGENCE_FLOW_PLAN_RESOLUTION` | Request planning |
| `INTELLIGENCE_FLOW_QUERY_DECORATOR` | Query processing |
| `INTELLIGENCE_PLATFORM_GLOBAL_KNOWLEDGE_SERVICE` | Knowledge base |
| `MAGIC_PAPER_ADAPTORS` | Writing Tools |

### Writing Tools

New text processing capabilities:
- Summarization
- Proofreading
- Tone adjustment
- Smart Reply

Tracked in logs via:
```bash
--predicate 'eventMessage CONTAINS "writing" OR eventMessage CONTAINS "summariz"'
```

### Genmoji

Generative emoji creation tracked in:
- `GenerativeFunctionMetrics_appleDiffusion_1_2`
- Logs with "genmoji" or "diffusion" mentions

---

## Changed from iOS 17

### Log Archive Version

```
OSArchiveVersion: 5 (unchanged)
OSLoggingSupportVersion: 1815.4 (updated)
```

### TCC.db Schema

No major schema changes from iOS 17. Same tables:
- `access`
- `policies`
- `active_policy`
- `access_overrides`
- `expired`

### PowerLog Tables

Many new tables added, existing tables unchanged. Table suffix convention:
- `_1_2` indicates schema version

### Process Count

Typical iOS 18 sysdiagnose:
- 450-500 unique processes
- 30-40 AI/ML related processes
- 100-150 XPC services

---

## iOS 18 Specific Queries

### Check AI Opt-In Status

```bash
log show --archive system_logs.logarchive \
    --predicate 'eventMessage CONTAINS[c] "apple intelligence" AND eventMessage CONTAINS "opt"' \
    --style json
```

### Count AI Events

```bash
log show --archive system_logs.logarchive \
    --predicate 'subsystem BEGINSWITH "com.apple.intelligence"' \
    --style json | grep -c '"timestamp"'
```

### Check GenerativeExperiences

```bash
cat logs/GenerativeExperiences/GEAvailability.log
```

### AI PowerLog Metrics

```bash
sqlite3 logs/powerlogs/*.PLSQL "
SELECT datetime(timestamp, 'unixepoch'), *
FROM GenerativeFunctionMetrics_OptIn_1_2
ORDER BY timestamp DESC
LIMIT 10"
```

---

## Known Issues

### Log Redaction

Apple Intelligence logs heavily redacted in production:
```
eventMessage: "Processing request for <private>"
```

### Model Downloads

Model catalog activity may not reflect actual downloads - check:
```bash
--predicate 'process == "nsurlsessiond" AND eventMessage CONTAINS "model"'
```

### Trial Configuration

Trial namespaces may show experimental (NCV=1) features in production builds.

---

## See Also

- [ios17.md](ios17.md) - iOS 17 notes
- [../subsystems/intelligence.md](../subsystems/intelligence.md) - AI subsystems
- [../processes/by-category/ai-ml.md](../processes/by-category/ai-ml.md) - AI processes
