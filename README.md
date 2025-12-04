# iOS Sysdiagnose Reference

A comprehensive reference for analyzing iOS sysdiagnose archives. Based on analysis of iOS 18.1 / 26.1 (Build 23B85) archives.

> **Version Note**: iOS uses dual versioning. Marketing version (18.1) vs internal version (26.1). Logs and sysdiagnose show the internal version (e.g., "iPhone OS 26.1").

## Quick Start

```bash
# Extract sysdiagnose
tar -xzf sysdiagnose_*.tar.gz

# Check structure
ls extracted_archive/

# Query unified logs
log show --archive extracted_archive/system_logs.logarchive \
    --predicate 'process == "SpringBoard"' \
    --style json

# Query TCC database
sqlite3 extracted_archive/logs/Accessibility/TCC.db \
    "SELECT service, client, auth_value FROM access"

# View crash reports
ls extracted_archive/crashes_and_spins/*.ips
```

---

## Documentation Structure

### [structure/](structure/) - Archive Layout
- **[overview.md](structure/overview.md)** - Top-level directory map
- **[system_logs.md](structure/system_logs.md)** - Unified logging (logarchive)
- **[crashes_and_spins.md](structure/crashes_and_spins.md)** - Crash reports

### [artifacts/](artifacts/) - Key Files
- **[ps.md](artifacts/ps.md)** - Process snapshot analysis
- **[spindump.md](artifacts/spindump.md)** - Stack sampling

### [network/](network/) - Network Data
- **[wifi.md](network/wifi.md)** - WiFi artifacts and history

### [privacy/](privacy/) - Privacy Artifacts
- **[tcc.md](privacy/tcc.md)** - Permission database (TCC.db)
- **[biome.md](privacy/biome.md)** - Behavioral intelligence

### [power/](power/) - Power & Telemetry
- **[powerlog.md](power/powerlog.md)** - PowerLog database (PLSQL)

### [subsystems/](subsystems/) - Log Subsystems
- **[index.md](subsystems/index.md)** - All com.apple.* subsystems
- **[intelligence.md](subsystems/intelligence.md)** - Apple Intelligence (iOS 18+)

### [processes/](processes/) - Process Reference
- **[index.md](processes/index.md)** - Process catalog
- **[by-category/ai-ml.md](processes/by-category/ai-ml.md)** - AI/ML processes

### [analysis/](analysis/) - Analysis Workflows
- **[common-queries.md](analysis/common-queries.md)** - Log query reference
- **[delta-comparison.md](analysis/delta-comparison.md)** - Comparing archives

### [formats/](formats/) - File Formats
- **[ips.md](formats/ips.md)** - Crash report format

### [databases/](databases/) - SQLite Databases
- **[overview.md](databases/overview.md)** - Database reference

---

## Common Tasks

### Find App Permissions

```bash
sqlite3 logs/Accessibility/TCC.db "
SELECT service, auth_value FROM access
WHERE client = 'com.example.app'
"
```

### Count Events by Process

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "locationd"' \
    --style json | grep -c '"timestamp"'
```

### Extract Crash Summary

```bash
for f in crashes_and_spins/*.ips; do
    head -1 "$f" | python3 -c "
import sys,json
d=json.load(sys.stdin)
print(f\"{d['timestamp']}: {d['app_name']}\")
"
done
```

### Compare Archives

```bash
# Event count comparison
for archive in baseline/* enabled/* disabled/*; do
    count=$(log show --archive "$archive/system_logs.logarchive" \
        --predicate 'process == "intelligenceplatformd"' \
        --style json 2>/dev/null | grep -c '"timestamp"')
    echo "$(basename $archive): $count"
done
```

---

## Key Forensic Artifacts

| Artifact | Location | Use Case |
|----------|----------|----------|
| Privacy permissions | `logs/Accessibility/TCC.db` | App data access |
| Unified logs | `system_logs.logarchive/` | System activity |
| Process list | `ps.txt` | Running processes |
| Crash reports | `crashes_and_spins/*.ips` | Crash analysis |
| WiFi history | `WiFi/Entity_*_Join.csv` | Network timeline |
| Power data | `logs/powerlogs/*.PLSQL` | Battery, usage |
| Trial config | `logs/Trial/*.log` | Feature flags |

---

## Tools

### Required
- `log` - Apple's unified log viewer (macOS)
- `sqlite3` - SQLite command-line
- `plutil` - Property list utility (macOS)

### Recommended
- `jq` - JSON processor
- `python3` - Scripting
- `ipsw` - iOS firmware tools

---

## iOS Version Notes

This reference is based on **iOS 18.1 / 26.1 (Build 23B85)**. Key differences from earlier versions:

### iOS 18+ Additions
- Apple Intelligence subsystems
- `GenerativeFunctionMetrics_*` PowerLog tables
- `logs/GenerativeExperiences/` directory
- Enhanced Trial namespace structure

### iOS 17 Compatibility
- Most structure remains the same
- Fewer AI-related artifacts
- Different PowerLog table set

---

## Contributing

To contribute additional documentation:

1. Follow existing file structure
2. Include practical examples
3. Reference actual sysdiagnose paths
4. Test commands against real archives

---

## References

- [Apple Unified Logging](https://developer.apple.com/documentation/os/logging)
- [Apple Security Research Device Program](https://security.apple.com/research-device/)
- [IPSW Tools](https://github.com/blacktop/ipsw)

---

## License

Documentation provided for educational and research purposes.
