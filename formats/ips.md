# IPS File Format

IPS (iOS Problem/Panic Signature) files are JSON-based crash and diagnostic reports. They're found in `crashes_and_spins/` and contain detailed information about process crashes, system panics, and other diagnostic events.

## File Structure

IPS files have two parts:

1. **Header line**: Single-line JSON with metadata
2. **Body**: Multi-line JSON with full report

```
{"app_name":"Photos","timestamp":"2025-11-22 19:29:43.00 -0500",...}
{
  "uptime": 440,
  "procRole": "Foreground",
  ...
}
```

## Header Fields

| Field | Type | Description |
|-------|------|-------------|
| `app_name` | string | Process name |
| `timestamp` | string | Crash time |
| `app_version` | string | App version |
| `build_version` | string | Build number |
| `bundleID` | string | Bundle identifier |
| `bug_type` | string | Report type code |
| `os_version` | string | OS version string |
| `platform` | int | Platform ID (2=iOS) |
| `is_first_party` | int | Apple app (1) or third-party (0) |
| `slice_uuid` | string | Binary UUID |
| `share_with_app_devs` | int | Sharing preference |
| `incident_id` | string | Unique incident ID |

## Bug Type Codes

| Code | Type | Description |
|------|------|-------------|
| `109` | Crash | Standard process crash |
| `210` | Spin | Unresponsive process |
| `288` | StackShot | Periodic stack sample |
| `298` | WatchdogViolation | Watchdog timeout |
| `308` | ExcUserFault | User-mode exception |
| `309` | ExcResource | Resource limit exceeded |
| `327` | AppLaunchFailure | App failed to launch |
| `142` | LowMemory | Low memory report |
| `179` | ThermalState | Thermal event |

## Body Structure

### Root Fields

```json
{
  "uptime": 440,                    // Seconds since boot
  "procRole": "Foreground",         // Process role
  "version": 2,                     // Report format version
  "userID": 501,                    // UID (501=mobile)
  "deployVersion": 210,             // Deployment version
  "modelCode": "iPhone16,1",        // Hardware model
  "coalitionID": 666,               // Coalition ID
  "pid": 435,                       // Process ID
  "procName": "Setup",              // Process name
  "procPath": "/Applications/Setup.app/Setup",
  "parentPid": 1,                   // Parent PID (usually 1=launchd)
  "coalitionName": "com.apple.purplebuddy",
  "translated": false,              // Rosetta translated
  "cpuType": "ARM-64"              // CPU architecture
}
```

### OS Version Block

```json
{
  "osVersion": {
    "isEmbedded": true,
    "train": "iPhone OS 26.1",
    "releaseType": "User",
    "build": "23B85"
  }
}
```

### Bundle Info

```json
{
  "bundleInfo": {
    "CFBundleIdentifier": "com.apple.purplebuddy",
    "CFBundleShortVersionString": "1.0.0",
    "CFBundleVersion": "1.0"
  }
}
```

### Exception Block

```json
{
  "exception": {
    "type": "EXC_CRASH",
    "codes": "0x0000000000000000, 0x0000000000000000",
    "subtype": "SIGABRT",
    "rawCodes": [0, 0],
    "signal": "SIGABRT"
  }
}
```

#### Exception Types

| Type | Description |
|------|-------------|
| `EXC_CRASH` | Process terminated abnormally |
| `EXC_BAD_ACCESS` | Memory access violation |
| `EXC_BAD_INSTRUCTION` | Illegal instruction |
| `EXC_ARITHMETIC` | Arithmetic exception |
| `EXC_GUARD` | Guard violation |
| `EXC_RESOURCE` | Resource limit exceeded |
| `EXC_BREAKPOINT` | Breakpoint/trace |

### Termination Block

```json
{
  "termination": {
    "namespace": "SIGNAL",
    "code": 6,
    "indicator": "Abort trap: 6",
    "byPid": 435,
    "byProc": "Setup",
    "flags": 0
  }
}
```

#### Termination Namespaces

| Namespace | Description |
|-----------|-------------|
| `SIGNAL` | Unix signal |
| `JETSAM` | Memory pressure kill |
| `FRONTBOARD` | UI timeout |
| `RUNNINGBOARD` | Lifecycle violation |
| `WEBKIT` | WebKit crash |
| `ASSERTIOND` | Assertion failure |

### Threads Array

```json
{
  "threads": [
    {
      "id": 8114,
      "name": "com.apple.main-thread",
      "frames": [
        {
          "imageOffset": 2900388204,
          "imageIndex": 0
        },
        ...
      ],
      "threadState": {
        "x": [{...}, ...],      // x0-x28 registers
        "lr": {"value": 0},     // Link register
        "pc": {"value": 0},     // Program counter
        "sp": {"value": 0},     // Stack pointer
        "fp": {"value": 0},     // Frame pointer
        "cpsr": {"value": 0}    // Status register
      },
      "queue": "com.apple.main-thread"
    }
  ],
  "faultingThread": 0           // Index of crashing thread
}
```

### Used Images Array

```json
{
  "usedImages": [
    {
      "base": 6442450944,       // Load address
      "size": 4811128948,       // Image size
      "uuid": "eba74a13-337f-38aa-9eec-231ffce9e8b6",
      "source": "S",            // S=SharedCache, P=Process
      "path": "/usr/lib/dyld"   // (optional)
    }
  ]
}
```

### Trial Info

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

---

## Parsing IPS Files

### Python Parser

```python
import json

def parse_ips(filepath):
    with open(filepath, 'r') as f:
        lines = f.readlines()

    header = json.loads(lines[0])
    body = json.loads(''.join(lines[1:]))

    return {
        'header': header,
        'body': body
    }

# Usage
report = parse_ips('crash.ips')
print(f"App: {report['header']['app_name']}")
print(f"Exception: {report['body']['exception']['type']}")
```

### Bash Extraction

```bash
# Extract header
head -1 crash.ips | python3 -m json.tool

# Extract body
tail -n +2 crash.ips | python3 -m json.tool

# Get app name
head -1 crash.ips | python3 -c "import sys,json; print(json.load(sys.stdin)['app_name'])"

# Get exception type
tail -n +2 crash.ips | python3 -c "import sys,json; print(json.load(sys.stdin).get('exception',{}).get('type','N/A'))"
```

### jq Queries

```bash
# Header info
head -1 crash.ips | jq '{app: .app_name, time: .timestamp, bug: .bug_type}'

# Body exception
tail -n +2 crash.ips | jq '.exception'

# Faulting thread
tail -n +2 crash.ips | jq '.threads[.faultingThread]'

# Used images
tail -n +2 crash.ips | jq '.usedImages[] | {uuid, base, source}'
```

---

## Symbolication

### Address Calculation

```
symbol_address = image_base + frame_offset
```

### Using atos

```bash
# Get image base and UUID from usedImages
# Symbol lookup:
atos -arch arm64e -o /path/to/binary -l 0x180000000 0x1804b6e28
```

### Using ipsw dyld

```bash
ipsw dyld symaddr /path/to/dyld_shared_cache 0x1804b6e28
```

---

## Common Analysis Tasks

### Find Exception Type

```bash
for f in crashes_and_spins/*.ips; do
    exc=$(tail -n +2 "$f" | python3 -c "import sys,json; print(json.load(sys.stdin).get('exception',{}).get('type','N/A'))" 2>/dev/null)
    echo "$(basename $f): $exc"
done
```

### Find EXC_GUARD Crashes

```bash
grep -l "EXC_GUARD" crashes_and_spins/*.ips
```

### Extract Crash Timeline

```bash
for f in crashes_and_spins/*.ips; do
    head -1 "$f" | python3 -c "
import sys,json
d=json.load(sys.stdin)
print(f\"{d['timestamp']}: {d['app_name']} ({d.get('bug_type','?')})\")"
done | sort
```

### Find Crashes by Process

```bash
for f in crashes_and_spins/*.ips; do
    app=$(head -1 "$f" | python3 -c "import sys,json; print(json.load(sys.stdin)['app_name'])" 2>/dev/null)
    echo "$app: $(basename $f)"
done | sort
```

---

## Report Notes

The `reportNotes` field contains diagnostic hints:

```json
{
  "reportNotes": [
    "PC register does not match crashing frame (0x0 vs 0x22CE0696C)"
  ]
}
```

Common notes:
- Register mismatch warnings
- Missing symbol information
- Truncation notices
- Collection warnings

---

## See Also

- [../structure/crashes_and_spins.md](../structure/crashes_and_spins.md) - Crash directory
- [../analysis/crash-analysis.md](../analysis/crash-analysis.md) - Analysis workflows
- [spindump.md](spindump.md) - Spindump format
