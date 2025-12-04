# Delta Comparison Analysis

Comparing sysdiagnose archives across time reveals changes in system behavior, permission grants, and activity patterns. This guide covers systematic delta analysis techniques.

## Archive Organization

### Naming Convention

```bash
sysdiagnose_2025.11.22_19-36-23-0500_iPhone-OS_iPhone_23B85  # Baseline
sysdiagnose_2025.11.25_23-05-46-0500_iPhone-OS_iPhone_23B85  # State A
sysdiagnose_2025.11.28_19-25-10-0500_iPhone-OS_iPhone_23B85  # State B
```

### Setup Variables

```bash
BASELINE="/path/to/sysdiagnose_2025.11.22_*/extracted"
ENABLED="/path/to/sysdiagnose_2025.11.25_*/extracted"
DISABLED="/path/to/sysdiagnose_2025.11.28_*/extracted"
```

---

## Quick Metrics Comparison

### Archive Sizes

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    du -sh "$archive"
    du -sh "$archive/system_logs.logarchive"
done
```

### File Counts

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    echo -n "Total files: "
    find "$archive" -type f | wc -l
    echo -n "TraceV3 files: "
    find "$archive/system_logs.logarchive" -name "*.tracev3" | wc -l
done
```

---

## Process Comparison

### Running Processes

```bash
# Extract process lists
awk 'NR>1 {print $NF}' "$BASELINE/ps.txt" | sort > /tmp/ps_baseline.txt
awk 'NR>1 {print $NF}' "$ENABLED/ps.txt" | sort > /tmp/ps_enabled.txt
awk 'NR>1 {print $NF}' "$DISABLED/ps.txt" | sort > /tmp/ps_disabled.txt

# New processes in ENABLED
echo "=== New in ENABLED ==="
comm -13 /tmp/ps_baseline.txt /tmp/ps_enabled.txt

# Removed in DISABLED
echo "=== Removed in DISABLED ==="
comm -23 /tmp/ps_enabled.txt /tmp/ps_disabled.txt
```

### Process Counts by Category

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    echo -n "AI processes: "
    awk 'NR>1 {print $NF}' "$archive/ps.txt" | \
        grep -iE "intelligence|siri|ml|model" | wc -l
    echo -n "Privacy processes: "
    awk 'NR>1 {print $NF}' "$archive/ps.txt" | \
        grep -iE "tcc|location|biome|parsec" | wc -l
done
```

---

## Unified Log Comparison

### Event Counts by Subsystem

```bash
count_events() {
    local archive="$1"
    local predicate="$2"
    log show --archive "$archive/system_logs.logarchive" \
        --predicate "$predicate" \
        --style json 2>/dev/null | grep -c '"timestamp"'
}

# Compare subsystems
for subsys in "com.apple.siri" "com.apple.springboard" "com.apple.TCC"; do
    echo "=== $subsys ==="
    for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
        count=$(count_events "$archive" "subsystem == \"$subsys\"")
        echo "$(basename $archive): $count"
    done
done
```

### Event Counts by Process

```bash
for proc in intelligenceplatformd modelcatalogd photoanalysisd locationd; do
    echo "=== $proc ==="
    for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
        count=$(count_events "$archive" "process == \"$proc\"")
        echo "$(basename $archive): $count"
    done
done
```

### Error Rate Comparison

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    echo -n "Errors: "
    count_events "$archive" 'messageType == error'
    echo -n "Faults: "
    count_events "$archive" 'messageType == fault'
done
```

---

## TCC Permission Comparison

### Permission Changes

```bash
# Extract permissions
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    name=$(basename "$archive")
    sqlite3 "$archive/logs/Accessibility/TCC.db" \
        "SELECT service || '|' || client || '|' || auth_value FROM access ORDER BY 1" \
        > "/tmp/tcc_$name.txt"
done

# Compare
echo "=== BASELINE → ENABLED ==="
diff /tmp/tcc_baseline.txt /tmp/tcc_enabled.txt

echo "=== ENABLED → DISABLED ==="
diff /tmp/tcc_enabled.txt /tmp/tcc_disabled.txt
```

### Permission Counts

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    echo -n "Total permissions: "
    sqlite3 "$archive/logs/Accessibility/TCC.db" "SELECT COUNT(*) FROM access"
    echo -n "Allowed: "
    sqlite3 "$archive/logs/Accessibility/TCC.db" "SELECT COUNT(*) FROM access WHERE auth_value=2"
    echo -n "Denied: "
    sqlite3 "$archive/logs/Accessibility/TCC.db" "SELECT COUNT(*) FROM access WHERE auth_value=0"
done
```

---

## PowerLog Comparison

### Table Record Counts

```bash
compare_powerlog_table() {
    local table="$1"
    echo "=== $table ==="
    for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
        db=$(find "$archive/logs/powerlogs" -name "*.PLSQL" | head -1)
        count=$(sqlite3 "$db" "SELECT COUNT(*) FROM $table" 2>/dev/null || echo "N/A")
        echo "$(basename $archive): $count"
    done
}

# Compare key tables
compare_powerlog_table "GenerativeFunctionMetrics_OptIn_1_2"
compare_powerlog_table "GenerativeFunctionMetrics_Summarization_1_2"
compare_powerlog_table "ANE_modelInference_1_2"
compare_powerlog_table "PLAppTimeService_Aggregate_AppRunTime"
```

---

## WiFi History Comparison

### Network Joins

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    join_file=$(find "$archive/WiFi" -name "*Join.csv.tgz" | head -1)
    if [ -n "$join_file" ]; then
        tar -xzf "$join_file" -O 2>/dev/null | wc -l
    fi
done
```

---

## Crash Report Comparison

### Crash Counts

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    echo -n "Crash reports: "
    ls "$archive/crashes_and_spins/"*.ips 2>/dev/null | wc -l
done
```

### New Crashes

```bash
# List crash files
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    ls "$archive/crashes_and_spins/"*.ips 2>/dev/null | xargs -n1 basename | sort
done | sort | uniq -c | sort -rn
```

---

## Trial Configuration Comparison

### Namespace Changes

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    cat "$archive/logs/Trial/trial-namespace-compatibility-versions.log" 2>/dev/null
done
```

### Experiment Enrollment

```bash
for archive in "$BASELINE" "$ENABLED" "$DISABLED"; do
    echo "=== $(basename $archive) ==="
    head -20 "$archive/logs/Trial/trial-experiment-info.log" 2>/dev/null
done
```

---

## Automated Delta Script

```bash
#!/bin/bash
# delta_analysis.sh - Compare sysdiagnose archives

BASELINE="$1"
COMPARE="$2"

echo "=== DELTA ANALYSIS ==="
echo "Baseline: $(basename $BASELINE)"
echo "Compare:  $(basename $COMPARE)"
echo ""

# Size comparison
echo "--- Archive Sizes ---"
echo "Baseline: $(du -sh "$BASELINE" | cut -f1)"
echo "Compare:  $(du -sh "$COMPARE" | cut -f1)"
echo ""

# Process count
echo "--- Process Counts ---"
echo "Baseline: $(awk 'NR>1' "$BASELINE/ps.txt" | wc -l)"
echo "Compare:  $(awk 'NR>1' "$COMPARE/ps.txt" | wc -l)"
echo ""

# Key metrics
echo "--- Key Event Counts ---"
for proc in intelligenceplatformd locationd photoanalysisd; do
    b_count=$(log show --archive "$BASELINE/system_logs.logarchive" \
        --predicate "process == \"$proc\"" --style json 2>/dev/null | grep -c '"timestamp"')
    c_count=$(log show --archive "$COMPARE/system_logs.logarchive" \
        --predicate "process == \"$proc\"" --style json 2>/dev/null | grep -c '"timestamp"')
    delta=$((c_count - b_count))
    pct=$(echo "scale=1; ($delta * 100) / ($b_count + 1)" | bc 2>/dev/null || echo "N/A")
    echo "$proc: $b_count → $c_count (${pct}%)"
done
```

---

## Visualization

### Simple Bar Chart (ASCII)

```bash
#!/bin/bash
# Generate ASCII bar chart of metrics

bar() {
    local val=$1
    local max=$2
    local width=50
    local filled=$((val * width / max))
    printf "["
    for ((i=0; i<filled; i++)); do printf "#"; done
    for ((i=filled; i<width; i++)); do printf " "; done
    printf "] %d\n" "$val"
}

echo "=== Event Counts ==="
echo -n "Baseline: "; bar 21497 100000
echo -n "Enabled:  "; bar 36188 100000
echo -n "Disabled: "; bar 12632 100000
```

---

## See Also

- [common-queries.md](common-queries.md) - Log query examples
- [timeline-construction.md](timeline-construction.md) - Building timelines
- [../structure/overview.md](../structure/overview.md) - Archive structure
