# iOS Processes Index

This index catalogs key iOS processes found in sysdiagnose archives, organized by function and forensic relevance.

## Process Locations

iOS processes reside in:

| Path | Type |
|------|------|
| `/sbin/` | Core system (launchd) |
| `/usr/libexec/` | System daemons |
| `/usr/sbin/` | System administration |
| `/System/Library/PrivateFrameworks/*/` | Framework daemons |
| `/System/Library/CoreServices/` | Core services |
| `/Applications/` | System apps |

---

## Critical System Processes

### Boot & Init

| PID | Process | Purpose |
|-----|---------|---------|
| 1 | `launchd` | Process manager, init |
| 33 | `logd` | Unified logging |
| 34 | `runningboardd` | Process lifecycle |
| 35 | `SpringBoard` | UI server |

### Security

| Process | Purpose | Forensic Value |
|---------|---------|----------------|
| `securityd` | Security framework | **Critical** |
| `keybagd` | Keychain | **Critical** |
| `amfid` | Code signing | **High** |
| `tccd` | Privacy permissions | **Critical** |
| `trustd` | Certificate trust | **High** |

### Networking

| Process | Purpose | Forensic Value |
|---------|---------|----------------|
| `configd` | Network configuration | **High** |
| `wifid` | WiFi management | **High** |
| `bluetoothd` | Bluetooth | **Medium** |
| `apsd` | Push notifications | **Medium** |
| `identityservicesd` | iMessage/FaceTime | **High** |
| `networkd` | Network stack | **Medium** |

---

## Privacy-Sensitive Processes

### Location

| Process | Purpose |
|---------|---------|
| `locationd` | Location services daemon |
| `routined` | Significant locations learning |
| `geod` | Geocoding/maps |

### Behavioral Data

| Process | Purpose |
|---------|---------|
| `biomed` | Biome behavioral data |
| `biomesyncd` | Biome iCloud sync |
| `parsecd` | Siri personalization |
| `parsec-fbf` | Parsec feedback |
| `coreduetd` | Duet scheduler |
| `duetexpertd` | Prediction engine |
| `contextstored` | Context storage |

### Health

| Process | Purpose |
|---------|---------|
| `healthd` | HealthKit daemon |
| `healthrecordsd` | Health records |
| `fitnessintelligenced` | Fitness ML |

### Privacy Enforcement

| Process | Purpose |
|---------|---------|
| `tccd` | TCC permission daemon |
| `privacyaccountingd` | Data access logging |
| `adprivacyd` | Ad privacy |
| `webprivacyd` | Web tracking prevention |

---

## AI/ML Processes

### Apple Intelligence (iOS 18+)

| Process | Purpose |
|---------|---------|
| `intelligenceplatformd` | AI orchestration |
| `IntelligencePlatformComputeService` | ML compute |
| `siriinferenced` | Siri on-device ML |
| `modelcatalogd` | Model catalog |
| `modelmanagerd` | Model lifecycle |
| `mlhostd` | ML hosting |

### Media Analysis

| Process | Purpose |
|---------|---------|
| `mediaanalysisd` | Media ML |
| `photoanalysisd` | Photo ML |
| `mediaremoted` | Media session control |
| `medialibraryd` | Media library |

### Siri

| Process | Purpose |
|---------|---------|
| `assistantd` | Siri main daemon |
| `siriknowledged` | Siri knowledge |
| `siriactionsd` | Shortcuts/actions |
| `sirittsd` | Text-to-speech |

### Search

| Process | Purpose |
|---------|---------|
| `corespotlightd` | Spotlight indexing |
| `spotlightknowledged` | Search knowledge |
| `searchpartyd` | Find My |

---

## App Lifecycle

| Process | Purpose |
|---------|---------|
| `runningboardd` | App state management |
| `frontboardd` | Foreground app |
| `backboardd` | Background/event handling |
| `SpringBoard` | Home screen, app switching |
| `installd` | App installation |
| `mobileinstallationd` | Mobile app install |

---

## Storage & Sync

| Process | Purpose |
|---------|---------|
| `mobileslideshow` | Photos |
| `photolibraryd` | Photo library |
| `backupd` | Local backup |
| `cloudd` | iCloud sync |
| `bird` | iCloud Drive |
| `nsurlsessiond` | URL downloads |

---

## Communications

| Process | Purpose |
|---------|---------|
| `imagent` | iMessage agent |
| `identityservicesd` | iMessage/FaceTime |
| `callservicesd` | Phone calls |
| `CommCenter` | Cellular |
| `facetimed` | FaceTime |

---

## Power & Thermal

| Process | Purpose |
|---------|---------|
| `powerd` | Power management |
| `thermalmonitord` | Thermal throttling |
| `batteryintelligenced` | Battery ML |

---

## Process Categories Quick Reference

### By User

```bash
# Root processes
grep "^root" ps.txt | wc -l

# Mobile (user) processes
grep "^mobile" ps.txt | wc -l
```

### By Path

```bash
# Framework daemons
grep "PrivateFrameworks" ps.txt | wc -l

# System daemons
grep "/usr/libexec/" ps.txt | wc -l

# XPC services
grep "\.xpc/" ps.txt | wc -l
```

---

## Finding Processes

### In ps.txt

```bash
# Find specific process
grep "processname" ps.txt

# Find by pattern
awk 'NR>1 {print $NF}' ps.txt | grep -iE "pattern"

# Count by path
awk 'NR>1 {print $NF}' ps.txt | cut -d/ -f1-4 | sort | uniq -c | sort -rn
```

### In Unified Logs

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "processname"'
```

### In Crash Reports

```bash
grep -l "processname" crashes_and_spins/*.ips
```

---

## Process Hierarchies

### Typical Parent Relationships

```
launchd (1)
├── logd (33)
├── runningboardd (34)
├── SpringBoard (35)
│   └── (app processes)
├── securityd
├── wifid
├── locationd
└── (other daemons)
```

### Finding Parent

```bash
awk -v pid=123 '$4 == pid {print "Parent:", $5}' ps.txt
```

---

## See Also

- [by-category/ai-ml.md](by-category/ai-ml.md) - AI/ML processes
- [by-category/privacy.md](by-category/privacy.md) - Privacy processes
- [by-category/network.md](by-category/network.md) - Network processes
- [../artifacts/ps.md](../artifacts/ps.md) - ps.txt analysis
