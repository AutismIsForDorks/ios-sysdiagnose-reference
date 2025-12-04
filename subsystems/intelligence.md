# Apple Intelligence Subsystems (iOS 18+)

Apple Intelligence is Apple's on-device AI platform introduced in iOS 18. This document covers the subsystems, processes, and logs relevant to forensic analysis.

## Core Subsystems

### com.apple.intelligenceplatform
Primary orchestration subsystem:
- Feature availability
- Model management
- Request routing

```bash
log show --archive system_logs.logarchive \
    --predicate 'subsystem BEGINSWITH "com.apple.intelligenceplatform"'
```

### com.apple.intelligenceplatform.compute
Compute service:
- ML inference
- Model execution
- Resource allocation

### com.apple.intelligenceplatform.core
Core platform:
- Initialization
- Configuration
- State management

## Related Subsystems

### com.apple.siri.inference
On-device Siri ML:
- Intent classification
- Entity extraction
- Query understanding

### com.apple.generativeexperiences
Generative AI features:
- Writing Tools
- Genmoji
- Image generation

### com.apple.summarization
Text summarization:
- Email summaries
- Notification summaries
- Web page summaries

### com.apple.photoanalysis
Photos intelligence:
- Memory curation
- Search indexing
- Face/scene recognition

### com.apple.semanticindex
Semantic search index:
- Content embedding
- Vector search
- Cross-app search

---

## Key Processes

### Core Platform

| Process | Path | Purpose |
|---------|------|---------|
| `intelligenceplatformd` | `/System/Library/PrivateFrameworks/IntelligencePlatformCore.framework/` | Main daemon |
| `IntelligencePlatformComputeService` | XPC service | ML compute |

### Model Management

| Process | Path | Purpose |
|---------|------|---------|
| `modelcatalogd` | `/System/Library/PrivateFrameworks/ModelCatalogRuntime.framework/` | Model catalog |
| `modelmanagerd` | `/usr/libexec/` | Model lifecycle |
| `mlhostd` | `/usr/libexec/` | ML hosting |

### Inference

| Process | Path | Purpose |
|---------|------|---------|
| `siriinferenced` | `/System/Library/PrivateFrameworks/SiriInference.framework/Support/` | Siri inference |
| `mediaanalysisd` | `/System/Library/PrivateFrameworks/MediaAnalysis.framework/` | Media ML |
| `photoanalysisd` | `/System/Library/PrivateFrameworks/PhotoAnalysis.framework/` | Photo ML |

---

## Log Analysis

### Opt-In Status

```bash
log show --archive system_logs.logarchive \
    --predicate 'eventMessage CONTAINS[c] "apple intelligence" AND eventMessage CONTAINS "opt"' \
    --style json
```

Look for messages like:
- "Current Apple Intelligence opt in state: false"
- "Apple Intelligence opt-in changed"

### Feature Availability

```bash
log show --archive system_logs.logarchive \
    --predicate 'eventMessage CONTAINS "GEAvailability" OR eventMessage CONTAINS "eligibility"' \
    --style json
```

### Model Downloads

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "modelcatalogd" OR process == "modelmanagerd"' \
    --style json
```

### Inference Activity

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "siriinferenced" OR eventMessage CONTAINS "inference"' \
    --style json
```

---

## Trial Namespaces

Apple Intelligence uses the Trial system for feature flags:

### Key Namespaces (iOS 18)

| Namespace | Purpose |
|-----------|---------|
| `AI_SAFETY_ADAPTORS` | Safety system configuration |
| `DEVICE_EXPERT_ADAPTORS` | Device-specific adaptations |
| `INTELLIGENCE_FLOW_PLAN_RESOLUTION` | Request planning |
| `INTELLIGENCE_FLOW_QUERY_DECORATOR` | Query processing |
| `INTELLIGENCE_PLATFORM_GLOBAL_KNOWLEDGE_SERVICE` | Knowledge base |
| `MAGIC_PAPER_ADAPTORS` | Writing Tools |

### Check Trial Config

```bash
cat logs/Trial/trial-namespace-compatibility-versions.log
```

### Maturity Levels (NCV)

| NCV | Maturity | Meaning |
|-----|----------|---------|
| 0 | Unknown | Not configured |
| 1 | Experimental | Beta/testing |
| 2 | Production | Stable release |

---

## PowerLog Tables

### Opt-In Tracking

```sql
SELECT * FROM GenerativeFunctionMetrics_OptIn_1_2;
```

### Feature Usage

```sql
-- Summarization
SELECT * FROM GenerativeFunctionMetrics_Summarization_1_2;

-- Image generation (Genmoji)
SELECT * FROM GenerativeFunctionMetrics_appleDiffusion_1_2;

-- Writing Tools
SELECT * FROM GenerativeFunctionMetrics_SmartReplySession_1_2;
```

### Model Loading

```sql
SELECT * FROM GenerativeFunctionMetrics_assetLoad_1_2;
```

### Neural Engine

```sql
-- ANE inference
SELECT * FROM ANE_modelInference_1_2;

-- ANE status
SELECT * FROM ANE_ANEStatus_1_2;

-- Model compilation
SELECT * FROM ANE_modelCompilation_1_2;
```

---

## GenerativeExperiences Logs

Check `logs/GenerativeExperiences/`:

```bash
ls logs/GenerativeExperiences/
# GEAvailability.log
```

`GEAvailability.log` contains:
- Feature eligibility checks
- Device capability assessment
- User consent status

---

## Privacy Indicators

### Data Processing

Look for:
- Semantic index building
- Photo analysis runs
- Spotlight indexing spikes

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "corespotlightd" OR process == "photoanalysisd"' \
    --style json | grep -c '"timestamp"'
```

### Consent State

The key forensic question: Is AI processing data despite opt-out?

Check for:
1. `opt in state: false` in logs
2. Concurrent privacy data access
3. Model download activity
4. Indexing increases

---

## Cross-Archive Analysis

Compare AI activity across sysdiagnoses:

```bash
for archive in baseline enabled disabled; do
    echo "=== $archive ==="

    # AI mentions
    echo -n "AI mentions: "
    log show --archive "$archive/system_logs.logarchive" \
        --predicate 'eventMessage CONTAINS[c] "apple intelligence"' \
        --style json 2>/dev/null | grep -c '"timestamp"'

    # Model activity
    echo -n "Model activity: "
    log show --archive "$archive/system_logs.logarchive" \
        --predicate 'process == "modelcatalogd"' \
        --style json 2>/dev/null | grep -c '"timestamp"'

    # Indexing
    echo -n "Indexing: "
    log show --archive "$archive/system_logs.logarchive" \
        --predicate 'process == "corespotlightd"' \
        --style json 2>/dev/null | grep -c '"timestamp"'
done
```

---

## Safety Systems

### com.apple.modelsafety

Model safety checks:
- Content filtering
- Output sanitization
- Guardrails

### Safety-Related Symbols

Look for in binary analysis:
- `ModelSafetyContext`
- `safetyCheckAlwaysPasses`
- `guardrailConfig`
- `ContentFilterResult`

---

## Forensic Checklist

### Determining AI State

1. Check opt-in status in logs
2. Count AI-related events
3. Check Trial namespace config
4. Analyze PowerLog AI tables
5. Review indexing activity
6. Compare across time periods

### Red Flags

- High AI activity with opt-in: false
- Model downloads after disable
- Indexing increases post-disable
- Safety check bypasses

---

## See Also

- [../privacy/biome.md](../privacy/biome.md) - Behavioral data
- [../power/powerlog.md](../power/powerlog.md) - AI metrics
- [../structure/logs/generative-experiences.md](../structure/logs/generative-experiences.md) - GenEx logs
- [siri.md](siri.md) - Siri subsystems
