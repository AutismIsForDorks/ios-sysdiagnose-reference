# ps.txt - Process Snapshot

The `ps.txt` file captures a snapshot of all running processes at the moment the sysdiagnose was generated. This is one of the most valuable artifacts for understanding system state.

## Format

Standard `ps` output with extended columns:

```
USER               UID PRSNA   PID  PPID        F  %CPU %MEM PRI NI      VSZ    RSS WCHAN    TT  STAT STARTED      TIME COMMAND
root                 0     -     1     0     4004   0.0  0.0   0  0        0      0 -        ??  ?s    7:22PM   0:00.00 /sbin/launchd
```

## Column Reference

| Column | Description |
|--------|-------------|
| `USER` | Process owner username |
| `UID` | Numeric user ID |
| `PRSNA` | Persona ID (200 = default, 199 = logd) |
| `PID` | Process ID |
| `PPID` | Parent process ID |
| `F` | Process flags (hex) |
| `%CPU` | CPU usage percentage |
| `%MEM` | Memory usage percentage |
| `PRI` | Priority |
| `NI` | Nice value |
| `VSZ` | Virtual memory size |
| `RSS` | Resident set size |
| `WCHAN` | Wait channel |
| `TT` | Terminal |
| `STAT` | Process state |
| `STARTED` | Start time |
| `TIME` | CPU time consumed |
| `COMMAND` | Full command path |

## Key Users

| User | UID | Purpose |
|------|-----|---------|
| `root` | 0 | System processes |
| `mobile` | 501 | User processes, apps |
| `_logd` | 272 | Logging daemon |
| `_accessoryupdater` | 278 | Accessory updates |
| `_spotlight` | 89 | Spotlight indexing |
| `_locationd` | 205 | Location services |

## Process States

| State | Meaning |
|-------|---------|
| `R` | Running |
| `S` | Sleeping (interruptible) |
| `D` | Disk wait (uninterruptible) |
| `T` | Stopped |
| `Z` | Zombie |
| `?` | Unknown |

State suffixes:
- `s` - Session leader
- `+` - Foreground process group
- `<` - High priority
- `N` - Low priority

## Early-Boot Processes (Low PIDs)

Processes with low PIDs (< 100) are critical system daemons started early in boot:

```
PID  Process              Purpose
1    launchd              System/user process manager
32   UserEventAgent       System event handling
33   logd                 Unified logging
34   runningboardd        Process lifecycle
35   SpringBoard          iOS UI server
36   peopled              Contacts framework
37   assistantd           Siri
41   axassetsd            Accessibility assets
45   configd              Network configuration
48   powerd               Power management
49   parsecd              Siri/Search personalization
52   biomed               Behavioral data (Biome)
53   wifid                WiFi management
55   keybagd              Keychain
64   identityservicesd    iMessage/FaceTime
```

## Process Categories

### AI/ML Processes

```
/System/Library/PrivateFrameworks/IntelligencePlatformCore.framework/intelligenceplatformd
/System/Library/PrivateFrameworks/IntelligencePlatformCompute.framework/XPCServices/IntelligencePlatformComputeService.xpc/IntelligencePlatformComputeService
/System/Library/PrivateFrameworks/SiriInference.framework/Support/siriinferenced
/System/Library/PrivateFrameworks/MediaAnalysis.framework/mediaanalysisd
/System/Library/PrivateFrameworks/ModelCatalogRuntime.framework/modelcatalogd
/usr/libexec/modelmanagerd
/usr/libexec/mlhostd
```

### Privacy-Sensitive Processes

```
/System/Library/PrivateFrameworks/TCC.framework/Support/tccd
/usr/libexec/locationd
/System/Library/PrivateFrameworks/BiomeStreams.framework/Support/biomed
/System/Library/PrivateFrameworks/CoreParsec.framework/parsecd
/System/Library/Frameworks/HealthKit.framework/healthd
/usr/libexec/adprivacyd
/System/Library/PrivateFrameworks/PrivacyAccounting.framework/Versions/A/Resources/privacyaccountingd
```

### Network Processes

```
/usr/sbin/wifid
/usr/sbin/bluetoothd
/System/Library/PrivateFrameworks/ApplePushService.framework/apsd
/System/Library/PrivateFrameworks/IDS.framework/identityservicesd.app/identityservicesd
/usr/libexec/configd
/usr/libexec/networkserviceproxy
```

### Security Processes

```
/usr/libexec/securityd
/usr/libexec/keybagd
/usr/libexec/amfid
/System/Library/Frameworks/Security.framework/XPCServices/TrustedPeersHelper.xpc/TrustedPeersHelper
```

## Analysis Techniques

### Count Unique Processes

```bash
awk 'NR>1 {print $NF}' ps.txt | sort | uniq | wc -l
```

### Find AI/ML Processes

```bash
awk 'NR>1 {print $NF}' ps.txt | grep -iE "intelligence|siri|ml|model|inference|neural|media|photo"
```

### Find High-Memory Processes

```bash
awk 'NR>1 && $7 > 1.0 {print $7, $NF}' ps.txt | sort -rn
```

### Find Processes by User

```bash
grep "^mobile" ps.txt | awk '{print $NF}' | sort | uniq
```

### Compare Process Lists Between Archives

```bash
# Extract process names
awk 'NR>1 {print $NF}' archive1/ps.txt | sort > /tmp/ps1.txt
awk 'NR>1 {print $NF}' archive2/ps.txt | sort > /tmp/ps2.txt

# Find new processes
comm -13 /tmp/ps1.txt /tmp/ps2.txt

# Find removed processes
comm -23 /tmp/ps1.txt /tmp/ps2.txt
```

### Find XPC Services

```bash
grep "\.xpc/" ps.txt | awk '{print $NF}'
```

## Forensic Indicators

### Suspicious Patterns

1. **Unknown processes** - Not in standard iOS paths
2. **High CPU** - Processes consuming excessive CPU
3. **Unusual users** - Processes running as unexpected UIDs
4. **Missing critical daemons** - Expected processes not running
5. **Duplicate processes** - Same daemon running multiple times

### Expected Counts (iOS 18)

| Category | Typical Count |
|----------|---------------|
| Total processes | 400-500 |
| Root processes | 80-120 |
| Mobile processes | 250-350 |
| XPC services | 100-150 |

## Related Files

| File | Additional Info |
|------|-----------------|
| `ps_thread.txt` | Per-thread details |
| `taskinfo.txt` | Task/port information |
| `jetsam_priority.txt` | Memory priority |
| `RunningBoard/RunningBoard_state.log` | Process lifecycle |

## See Also

- [taskinfo.md](taskinfo.md) - Task details
- [../processes/index.md](../processes/index.md) - Process reference
- [../processes/by-category/ai-ml.md](../processes/by-category/ai-ml.md) - AI/ML processes
