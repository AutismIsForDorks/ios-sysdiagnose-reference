# spindump-nosymbols.txt - System Stack Samples

The spindump captures stack traces across all processes over a short sampling period. It's essential for performance analysis and identifying stuck or CPU-intensive processes.

## Header Information

```
Date/Time:        2025-11-22 19:36:24.011 -0500
End time:         2025-11-22 19:36:26.009 -0500
OS Version:       iPhone OS 26.1 (Build 23B85)
Architecture:     arm64e
Report Version:   67

Data Source:      Stackshots
Shared Cache:     EBA74A13-337F-38AA-9EEC-231FFCE9E8B6 slid base address 0x18039c000, slide 0x39c000
```

### Key Header Fields

| Field | Description |
|-------|-------------|
| `Date/Time` | Sample start time |
| `End time` | Sample end time |
| `Duration` | Total sampling period |
| `Steps` | Number of sample iterations |
| `Shared Cache` | dyld shared cache UUID and slide |
| `Hardware model` | Device identifier |
| `Active cpus` | Number of active CPU cores |
| `Memory size` | Total RAM |

## System Metrics

```
Duration:         2.00s
Steps:            8 (250ms sampling interval)

Hardware model:   iPhone16,1
Active cpus:      6
Memory size:      7.50 GB
HW page size:     16384
VM page size:     16384
Shared cache residency: 23.84% (1343.59 MB / 5636.75 MB)

Time Since Boot:  848s
Time Awake Since Boot: 848s
Time Since Wake:  n/a (machine hasn't slept)

Total CPU Time:   2.648s (2.9G cycles, 4.1G instructions, 0.71c/i)
Advisory levels:  Battery -> 1, User -> 3, ThermalPressure -> 0, Combined -> 1
Free disk space:  103.65 GB/118.76 GB, low space threshold 150 MB
Vnodes Available: 74.98% (20994/28000, 14000 allocated, 14000 soft limit)
```

### Advisory Levels

| Level | Meaning |
|-------|---------|
| `Battery -> 1` | Battery charge level indicator |
| `User -> 3` | User activity level |
| `ThermalPressure -> 0` | Thermal throttling (0=none) |
| `Combined -> 1` | Overall system pressure |

## Process Entries

Each process section contains:

```
Process:          AAUIFollowUpExtension [760] (suspended)
UUID:             6EAD3AFE-1E1F-36F3-B55B-EC1E7E6058E6
Path:             /System/Library/PrivateFrameworks/AppleAccountUI.framework/PlugIns/AAUIFollowUpExtension.appex/AAUIFollowUpExtension
Identifier:       com.apple.AppleAccountUI.AAUIFollowUpExtension
Version:          1.0 (1)
Shared Cache:     EBA74A13-337F-38AA-9EEC-231FFCE9E8B6 slid base address 0x18039c000, slide 0x39c000
Architecture:     arm64e
Parent:           launchd [1]
Responsible:      followupd [543]
RunningBoard Mgd: Yes
UID:              501
Memory Limit:     400MB
Jetsam Priority:  0
Footprint:        4848 KB
Time Since Fork:  392s
Num samples:      8 (1-8)
Note:             Suspended for 8 samples
Num threads:      7
```

### Process Fields

| Field | Description |
|-------|-------------|
| `Process` | Name, PID, and state |
| `UUID` | Binary UUID for symbolication |
| `Path` | Full executable path |
| `Identifier` | Bundle identifier |
| `Parent` | Parent process |
| `Responsible` | Process responsible for lifecycle |
| `RunningBoard Mgd` | Managed by RunningBoard (Yes/No) |
| `Memory Limit` | Jetsam memory limit |
| `Jetsam Priority` | Memory pressure kill priority |
| `Footprint` | Current memory footprint |
| `Time Since Fork` | Process age |
| `Num samples` | Times this process was sampled |

## Thread Stacks

```
  Thread 0x42d1    DispatchQueue "com.apple.main-thread"(1)    8 samples (1-8)    priority 4 (base 4)    last ran 324.825s ago
  8  ??? (dyld + 20008) [0x1804b6e28]
    8  ??? (Foundation + 1407108) [0x180c4f884]
      8  ??? (ExtensionFoundation + 207408) [0x184b9fa30]
        8  ??? (PlugInKit + 103196) [0x1c6c4731c]
```

### Thread Information

| Field | Description |
|-------|-------------|
| `Thread 0x42d1` | Thread ID (hex) |
| `DispatchQueue` | GCD queue name |
| `8 samples (1-8)` | Sampled in all 8 iterations |
| `priority 4` | Thread priority |
| `last ran` | Time since last CPU time |

### Stack Format

Without symbols:
```
8  ??? (dyld + 20008) [0x1804b6e28]
```

- `8` - Sample count (times seen at this frame)
- `???` - Unknown symbol (no symbols)
- `dyld + 20008` - Library and offset
- `[0x1804b6e28]` - Absolute address

## Launchd Throttled Processes

```
Launchd throttled processes:
  stickersd throttled after exit(): throttled samples 1-8
```

These are processes that launchd rate-limited due to rapid restart cycles.

## Analysis Techniques

### Find Heavy CPU Processes

Look for processes with high sample counts and running threads:

```bash
grep -E "^Process:|Total CPU Time:" spindump-nosymbols.txt | \
    grep -B1 "CPU Time" | grep "Process:"
```

### Find Stuck Threads

Threads with identical stacks across all samples may be stuck:

```bash
# Threads sampled 8/8 times (in a 8-step dump)
grep "8 samples (1-8)" spindump-nosymbols.txt
```

### Extract All Processes

```bash
grep "^Process:" spindump-nosymbols.txt | \
    sed 's/Process: *\([^ ]*\) .*/\1/' | sort
```

### Find Suspended Processes

```bash
grep "(suspended)" spindump-nosymbols.txt
```

### Find High Memory Processes

```bash
grep -E "^Process:|Footprint:" spindump-nosymbols.txt | \
    grep -B1 "Footprint:" | paste - - | \
    sort -t: -k4 -n | tail -20
```

## Symbolication

To symbolicate the stacks:

1. **Get shared cache UUID**: From header `Shared Cache:` line
2. **Extract shared cache**: From device or IPSW
3. **Use atos**:
```bash
atos -arch arm64e -o /path/to/dyld_shared_cache_arm64e \
    -l 0x18039c000 0x1804b6e28
```

Or use `ipsw dyld` tools:
```bash
ipsw dyld symaddr /path/to/dsc 0x1804b6e28
```

## Common Patterns

### XPC Service Waiting

```
8  ??? (libxpc.dylib + 132696) [0x19761f658]
  8  ??? (libxpc.dylib + 132216) [0x19761f478]
    8  ??? (libxpc.dylib + 122380) [0x19761ce0c]
```

XPC services often block waiting for messages - this is normal.

### RunLoop Waiting

```
8  ??? (CoreFoundation + 619748) [0x180534de4]
  8  ??? (CoreFoundation + 634140) [0x1805383ac]
    8  ??? (libsystem_kernel.dylib + 3588) [0x1977b2e04]
```

Main threads often wait in the run loop - normal for UI processes.

### Mach Port Wait

```
8  ??? (libsystem_kernel.dylib + 5280) [0x1977b34a0]
```

Threads waiting on mach_msg - inter-process communication wait.

## Related Files

| File | Description |
|------|-------------|
| `microstackshots/` | Lighter-weight periodic samples |
| `tailspin-info.txt` | Tailspin trace metadata |
| `taskinfo.txt` | Detailed task/thread info |

## See Also

- [microstackshots.md](microstackshots.md) - Continuous sampling
- [../formats/spindump.md](../formats/spindump.md) - Format details
- [../analysis/performance.md](../analysis/performance.md) - Performance analysis
