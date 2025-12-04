# iOS Sysdiagnose Structure Overview

A sysdiagnose archive contains a comprehensive snapshot of iOS system state, logs, and diagnostic data. This document maps the top-level structure based on iOS 18.1 / 26.1 (Build 23B85).

> **Version Note**: Sysdiagnose filenames and logs show iOS internal version (26.1), not marketing version (18.1).

## Archive Naming Convention

```
sysdiagnose_YYYY.MM.DD_HH-MM-SS-TZOFFSET_iPhone-OS_DEVICE_BUILD.tar.gz
```

Example: `sysdiagnose_2025.11.22_19-36-23-0500_iPhone-OS_iPhone_23B85`

- `YYYY.MM.DD_HH-MM-SS` - Capture timestamp
- `-0500` - Timezone offset (EST in this case)
- `iPhone-OS` - Platform
- `iPhone` - Device type
- `23B85` - Build number (iOS 18.1)

---

## Top-Level Directory Structure

### Core System Logs

| File/Directory | Description | Forensic Value |
|----------------|-------------|----------------|
| `system_logs.logarchive/` | Unified logging system archive (tracev3 files) | **Critical** - Primary log source |
| `logs/` | Categorized subsystem logs | **High** - Per-service diagnostics |
| `transparency.log` | Transparency daemon activity | **High** - Code signing, entitlements |

### Process State

| File | Description | Forensic Value |
|------|-------------|----------------|
| `ps.txt` | Process snapshot at capture time | **Critical** - Running processes, PIDs, users |
| `ps_thread.txt` | Per-thread process information | **High** - Thread states |
| `taskinfo.txt` | Detailed task information | **High** - Memory, ports per process |
| `jetsam_priority.txt` | Memory pressure priority list | **Medium** - Jetsam ordering |
| `jetsam_priority.csv` | Same as above, CSV format | **Medium** |
| `taskSummary.csv` | Task summary statistics | **Medium** |

### Crash and Performance

| File/Directory | Description | Forensic Value |
|----------------|-------------|----------------|
| `crashes_and_spins/` | Crash reports (.ips) and spin reports | **Critical** - Crash analysis |
| `spindump-nosymbols.txt` | Stack samples across all processes | **High** - Performance analysis |
| `microstackshots/` | Lightweight stack samples | **Medium** - CPU profiling |
| `tailspin-info.txt` | Tailspin tracing info | **Medium** |

### System Configuration

| File | Description | Forensic Value |
|------|-------------|----------------|
| `sysctl.txt` | Kernel parameters | **Medium** - System config |
| `mount.txt` | Mounted filesystems | **Medium** - Storage state |
| `disks.txt` | Disk information | **Low** |
| `apfs_stats.txt` | APFS filesystem statistics | **Low** |
| `vm_stat.txt` | Virtual memory statistics | **Low** |
| `zprint.txt` | Zone allocator statistics | **Low** |

### Hardware State

| File/Directory | Description | Forensic Value |
|----------------|-------------|----------------|
| `ioreg/` | I/O Registry dump | **Medium** - Hardware tree |
| `hidutil.plist` | HID device information | **Low** |
| `smcDiagnose.txt` | SMC (System Management) data | **Low** |
| `hpmDiagnose.txt` | HPM diagnostics | **Low** |
| `codecctl.txt` | Audio codec state | **Low** |

### Security

| File | Description | Forensic Value |
|------|-------------|----------------|
| `security-sysdiagnose.txt` | Security state summary | **High** - Keychain, entitlements |
| `ckksctl_status.txt` | CloudKit Keychain Sync status | **High** - iCloud keychain |
| `pcsstatus.txt` | Protected Cloud Storage status | **Medium** |
| `otctl_status.txt` | Octagon Trust status | **Medium** |

### Network

| File/Directory | Description | Forensic Value |
|----------------|-------------|----------------|
| `WiFi/` | WiFi diagnostics, known networks | **Critical** - Network history |
| `remotectl_dumpstate.txt` | Remote control state | **Medium** |

### Service State

| File/Directory | Description | Forensic Value |
|----------------|-------------|----------------|
| `RunningBoard/` | Process lifecycle management | **High** - App states |
| `brctl/` | iCloud Drive container state | **Medium** |
| `FileProvider/` | File provider extension state | **Low** |
| `Preferences/` | System preferences | **Medium** |
| `summaries/` | Various summary reports | **Medium** |
| `Personalization/` | Personalization data | **Low** |
| `ASPSnapshots/` | App Store snapshots | **Low** |
| `TimezoneDB/` | Timezone database | **Low** |

### Metadata

| File | Description |
|------|-------------|
| `README.txt` | Sysdiagnose generation info |
| `sysdiagnose.log` | Sysdiagnose capture log |
| `oslog_archive_error.log` | Log archive errors |

---

## Directory Sizes (Typical iOS 18.1)

| Component | Typical Size | Notes |
|-----------|--------------|-------|
| `system_logs.logarchive/` | 50-150 MB | Varies with log activity |
| `crashes_and_spins/` | 1-50 MB | Depends on crash history |
| `logs/powerlogs/` | 10-40 MB | PowerLog database |
| `WiFi/` | 5-20 MB | Includes CSVs |
| `microstackshots/` | 1-10 MB | Sampling data |
| `spindump-nosymbols.txt` | 1-5 MB | Full system stack sample |

**Total archive size:** 100-500 MB (compressed)

---

## Quick Reference: Finding Common Data

### User Activity
- `logs/powerlogs/*.PLSQL` - App usage, screen time
- `WiFi/Entity_*_Join.csv` - Network connections
- `logs/Accessibility/TCC.db` - Permission grants

### AI/ML Activity (iOS 18+)
- `logs/GenerativeExperiences/` - Apple Intelligence status
- `logs/ModelCatalog/` - ML model state
- `logs/ModelManager/` - Model management
- `logs/Trial/` - Feature flag experiments

### Security Events
- `security-sysdiagnose.txt` - Keychain state
- `ckksctl_status.txt` - iCloud Keychain sync
- `logs/Accessibility/TCC.db` - Privacy permissions

### Network History
- `WiFi/com.apple.wifi.plist` - Current WiFi config
- `WiFi/Entity_*_Join.csv` - Join history
- `WiFi/Entity_*_Network.csv` - Known networks

---

## See Also

- [system_logs.md](system_logs.md) - Detailed logarchive structure
- [crashes_and_spins.md](crashes_and_spins.md) - Crash report analysis
- [../artifacts/ps.md](../artifacts/ps.md) - Process snapshot analysis
