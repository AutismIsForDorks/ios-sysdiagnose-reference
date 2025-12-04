# crashes_and_spins/ Directory

Contains crash reports (.ips files), spin reports, and stack shot captures for processes that terminated abnormally or exhibited performance issues.

## File Types

| Extension | Type | Description |
|-----------|------|-------------|
| `.ips` | JSON | Crash/exception reports |
| `stacks-*.ips` | JSON | Periodic stack samples |
| `OTAUpdate-*.ips` | JSON | OTA update crashes |
| `ExcUserFault_*.ips` | JSON | User-fault exceptions |

## IPS File Format

IPS files are JSON with a single-line header followed by a JSON body:

```json
{"is_simulated":0,"app_name":"Photos","timestamp":"2025-11-22 19:29:43.00 -0500",...}
{
  "uptime": 440,
  "procRole": "Foreground",
  "version": 2,
  ...
}
```

### Header Fields

| Field | Description |
|-------|-------------|
| `app_name` | Process name |
| `timestamp` | Crash time |
| `bundleID` | Bundle identifier |
| `bug_type` | Report type code |
| `os_version` | iOS version string |
| `build_version` | Build number |
| `is_first_party` | Apple process (1) or third-party (0) |

### Bug Type Codes

| Code | Type | Description |
|------|------|-------------|
| `109` | Crash | Standard crash report |
| `210` | Spin | Unresponsive process |
| `298` | WatchdogViolation | Watchdog timeout |
| `308` | ExcUserFault | User-mode exception |
| `309` | ExcResource | Resource limit exceeded |
| `327` | AppLaunchFailure | Launch failure |

### Body Structure

```json
{
  "uptime": 440,                    // Seconds since boot
  "procRole": "Foreground",         // Process role
  "version": 2,                     // Report format version
  "userID": 501,                    // UID (501 = mobile)
  "modelCode": "iPhone16,1",        // Hardware model
  "osVersion": {
    "train": "iPhone OS 26.1",
    "build": "23B85",
    "releaseType": "User"
  },
  "captureTime": "2025-11-22 19:29:43.4815 -0500",
  "incident": "41030059-1E11-4EF3-B9C3-EDDF13AFD93F",
  "pid": 435,
  "procName": "Setup",
  "procPath": "/Applications/Setup.app/Setup",
  "bundleInfo": {
    "CFBundleIdentifier": "com.apple.purplebuddy",
    "CFBundleVersion": "1.0"
  },
  "exception": {
    "type": "EXC_CRASH",
    "codes": "0x0000000000000000, 0x0000000000000000",
    "subtype": "SIGABRT"
  },
  "termination": {
    "namespace": "SIGNAL",
    "code": 6,
    "indicator": "Abort trap: 6"
  },
  "faultingThread": 0,
  "threads": [...],
  "usedImages": [...],
  "trialInfo": {...}
}
```

## Exception Types

### Common Exceptions

| Type | Description | Typical Cause |
|------|-------------|---------------|
| `EXC_CRASH` | Unhandled signal | SIGABRT, SIGKILL |
| `EXC_BAD_ACCESS` | Memory access violation | Null pointer, use-after-free |
| `EXC_BAD_INSTRUCTION` | Invalid instruction | Undefined behavior |
| `EXC_GUARD` | Guard exception | Protected resource access |
| `EXC_RESOURCE` | Resource limit | CPU/memory exceeded |

### EXC_GUARD Subtypes

| Subtype | Description |
|---------|-------------|
| `GUARD_TYPE_MACH_PORT` | Mach port violation |
| `GUARD_TYPE_FD` | File descriptor violation |
| `GUARD_TYPE_USER` | User-defined guard |

### Termination Namespaces

| Namespace | Description |
|-----------|-------------|
| `SIGNAL` | Unix signal |
| `JETSAM` | Memory pressure kill |
| `FRONTBOARD` | UI timeout |
| `WEBKIT` | WebKit crash |

## Thread Analysis

### Thread State

```json
{
  "id": 8114,
  "frames": [
    {
      "imageOffset": 2900388204,
      "imageIndex": 0
    },
    ...
  ],
  "threadState": {
    "x": [...],           // ARM64 registers x0-x28
    "lr": {"value": 0},   // Link register
    "pc": {"value": 0},   // Program counter
    "sp": {"value": 0},   // Stack pointer
    "fp": {"value": 0},   // Frame pointer
    "cpsr": {"value": 0}  // Status register
  }
}
```

### Symbolication

Frames contain offsets, not symbols. To symbolicate:

1. Find the image in `usedImages`:
```json
{
  "size": 4811128948,
  "base": 6442450944,
  "uuid": "eba74a13-337f-38aa-9eec-231ffce9e8b6",
  "source": "S"  // S=SharedCache, P=Process
}
```

2. Calculate address: `base + imageOffset`

3. Use atos or Xcode:
```bash
atos -arch arm64e -o dyld_shared_cache -l 0x180000000 0x18AD5FE6C
```

## Trial Info

Crash reports include active experiments:

```json
{
  "trialInfo": {
    "rollouts": [
      {
        "rolloutId": "6081ed9716bb6d61d81d5014",
        "factorPackIds": ["691e000129ea8563ffecf888"],
        "deploymentId": 240003868
      }
    ],
    "experiments": []
  }
}
```

This helps identify if experimental features caused the crash.

## Stack Shots (stacks-*.ips)

Periodic system-wide stack samples:

```json
{
  "bug_type": "288",
  "captureTime": "2025-11-22 19:36:22",
  "reason": "Periodic stack shot",
  "processes": [
    {
      "pid": 1,
      "procName": "launchd",
      "threads": [...]
    },
    ...
  ]
}
```

Useful for:
- System-wide performance analysis
- Finding CPU-heavy processes
- Detecting stuck threads

## Forensic Analysis

### Security-Relevant Crashes

Look for:
- `EXC_GUARD` - May indicate exploit attempts
- `EXC_BAD_ACCESS` - Memory corruption
- Crashes in security-sensitive processes (tccd, securityd)
- Repeated crashes in same process

### Privacy-Relevant Crashes

Check crashes in:
- `locationd` - Location services
- `tccd` - TCC/privacy daemon
- `biomed` - Behavioral data
- `parsecd` - Siri/Search data

### XPC Crashes

Look for XPC-related frames:
```bash
grep -l "xpc" crashes_and_spins/*.ips
```

May indicate IPC issues or sandbox violations.

## Parsing Examples

### Extract Crash Summaries

```bash
for f in crashes_and_spins/*.ips; do
  echo "=== $(basename $f) ==="
  head -1 "$f" | python3 -c "import sys,json; d=json.load(sys.stdin); print(f\"{d['app_name']} - {d.get('bug_type','unknown')}\")"
done
```

### Find EXC_GUARD Crashes

```bash
grep -l "EXC_GUARD" crashes_and_spins/*.ips
```

### Extract Exception Types

```bash
for f in crashes_and_spins/*.ips; do
  tail -n +2 "$f" | python3 -c "
import sys,json
try:
    d=json.load(sys.stdin)
    exc=d.get('exception',{})
    print(f\"{d.get('procName','?')}: {exc.get('type','?')} - {exc.get('subtype','?')}\")
except: pass
"
done
```

## Retention

- Crash reports retained for ~30 days
- High-priority crashes may persist longer
- OTA update crashes always preserved
- User-reported crashes flagged

## See Also

- [../formats/ips.md](../formats/ips.md) - IPS format details
- [../analysis/crash-analysis.md](../analysis/crash-analysis.md) - Analysis workflows
- [../processes/index.md](../processes/index.md) - Process reference
