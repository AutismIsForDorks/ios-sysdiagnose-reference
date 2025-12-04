# logs/Trial/ - Feature Flag System

The Trial directory contains Apple's feature flag and A/B experimentation system data. This is crucial for understanding which features are enabled and their maturity level.

## Directory Contents

| File | Description |
|------|-------------|
| `trial.log` | Main trial daemon log |
| `trial-experiment-info.log` | Active experiment enrollments |
| `trial-mixed-experiment-info.log` | Mixed experiment data |
| `trial-server-side-experiment-info.log` | Server-controlled experiments |
| `trial-namespace-compatibility-versions.log` | Namespace maturity versions |
| `trial-rollout-info.log` | Feature rollout status |

---

## Namespace Compatibility Versions (NCV)

The `trial-namespace-compatibility-versions.log` file shows feature maturity:

### NCV Values

| NCV | Maturity | Meaning |
|-----|----------|---------|
| 0 | Unknown | Not configured |
| 1 | Experimental | Beta/testing, may be unstable |
| 2 | Production | Stable, fully released |

### Example Output

```
AI_SAFETY_ADAPTORS: NCV=1
DEVICE_EXPERT_ADAPTORS: NCV=1
INTELLIGENCE_FLOW_PLAN_RESOLUTION: NCV=1
INTELLIGENCE_FLOW_QUERY_DECORATOR: NCV=1
INTELLIGENCE_PLATFORM_GLOBAL_KNOWLEDGE_SERVICE: NCV=1
MAGIC_PAPER_ADAPTORS: NCV=1
```

### Parsing NCV

```bash
cat logs/Trial/trial-namespace-compatibility-versions.log | \
    grep -E "NCV=[0-9]" | sort
```

---

## Key Namespaces (iOS 18)

### Apple Intelligence

| Namespace | Purpose |
|-----------|---------|
| `AI_SAFETY_ADAPTORS` | Safety guardrails configuration |
| `DEVICE_EXPERT_ADAPTORS` | Device-specific model adaptations |
| `INTELLIGENCE_FLOW_PLAN_RESOLUTION` | Request planning |
| `INTELLIGENCE_FLOW_QUERY_DECORATOR` | Query enhancement |
| `INTELLIGENCE_PLATFORM_GLOBAL_KNOWLEDGE_SERVICE` | Knowledge base |
| `MAGIC_PAPER_ADAPTORS` | Writing Tools |

### System Features

| Namespace | Purpose |
|-----------|---------|
| `SIRI_*` | Siri features |
| `PHOTOS_*` | Photos app features |
| `MESSAGES_*` | Messages features |
| `SAFARI_*` | Safari features |

---

## Experiment Info

### trial-experiment-info.log

Shows active A/B experiments:

```
no experiments
```

Or if enrolled:
```
experiment_id: abc123
treatment: variant_a
enrollment_date: 2025-11-22
```

### trial-server-side-experiment-info.log

Server-controlled experiments (CloudKit-based).

---

## Rollout Info

### trial-rollout-info.log

Feature rollout percentages and status:

```
rollout_id: 6081ed9716bb6d61d81d5014
deployment_id: 240003868
percentage: 100
```

---

## Forensic Analysis

### Check Feature Maturity

```bash
# Find experimental features (NCV=1)
grep "NCV=1" logs/Trial/trial-namespace-compatibility-versions.log

# Find production features (NCV=2)
grep "NCV=2" logs/Trial/trial-namespace-compatibility-versions.log
```

### Compare Across Archives

```bash
for archive in baseline enabled disabled; do
    echo "=== $archive ==="
    cat "$archive/logs/Trial/trial-namespace-compatibility-versions.log" | \
        grep -c "NCV=1" | xargs echo "Experimental:"
    cat "$archive/logs/Trial/trial-namespace-compatibility-versions.log" | \
        grep -c "NCV=2" | xargs echo "Production:"
done
```

### Cross-Reference with Logs

```bash
log show --archive system_logs.logarchive \
    --predicate 'subsystem == "com.apple.trial"' \
    --style json
```

---

## Privacy Implications

Trial data reveals:
- Features being tested on device
- Experiment enrollments
- Feature flag states

This can indicate:
- Beta feature usage
- Targeted feature rollouts
- A/B test participation

---

## See Also

- [index.md](index.md) - Logs directory index
- [../../subsystems/intelligence.md](../../subsystems/intelligence.md) - AI namespaces
- [../../versions/ios18.md](../../versions/ios18.md) - iOS 18 Trial changes
