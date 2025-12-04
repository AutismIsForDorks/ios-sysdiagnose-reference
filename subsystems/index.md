# iOS Subsystems Index

Unified logging subsystems (`com.apple.*`) organize log messages by component. This index lists key subsystems for forensic analysis.

## Subsystem Hierarchy

```
com.apple.{component}.{subcomponent}
```

Example: `com.apple.siri.inference`

## Finding Subsystems in Logs

```bash
# List all subsystems in archive
log show --archive system_logs.logarchive --style json 2>/dev/null | \
    grep -o '"subsystem" *: *"[^"]*"' | sort | uniq -c | sort -rn

# Filter by subsystem
log show --archive system_logs.logarchive \
    --predicate 'subsystem == "com.apple.springboard"'
```

---

## Privacy & Security

### com.apple.TCC
**Transparency, Consent, and Control**
- Permission decisions
- App authorization
- Privacy prompts

```bash
--predicate 'subsystem == "com.apple.TCC"'
```

### com.apple.security
**Security framework**
- Keychain access
- Code signing
- Entitlements

### com.apple.securityd
**Security daemon**
- Key operations
- Certificate validation

### com.apple.locationd
**Location services**
- GPS/WiFi location
- Geofencing
- Significant location changes

### com.apple.privacyaccounting
**Privacy accounting**
- Data access logging
- Privacy reports

---

## AI / Machine Learning

### com.apple.intelligenceplatform
**Apple Intelligence core**
- AI orchestration
- Feature flags
- Model management

```bash
--predicate 'subsystem BEGINSWITH "com.apple.intelligence"'
```

### com.apple.siri
**Siri umbrella**
- Voice recognition
- Intent handling
- Suggestions

### com.apple.siri.inference
**Siri ML inference**
- On-device models
- Query processing

### com.apple.photos.intelligence
**Photos ML**
- Face detection
- Scene classification
- Object recognition

### com.apple.photoanalysis
**Photo analysis**
- Image indexing
- Feature extraction

### com.apple.spotlight
**Spotlight search**
- Content indexing
- Query handling

### com.apple.parsec
**Parsec personalization**
- Search ranking
- Suggestions

### com.apple.biome
**Behavioral intelligence**
- Usage patterns
- Predictions

### com.apple.duet
**Duet scheduler**
- Background task scheduling
- Power-aware execution

---

## User Interface

### com.apple.springboard
**SpringBoard (Home screen)**
- App launches
- Notifications
- UI state

```bash
--predicate 'subsystem == "com.apple.springboard"'
```

### com.apple.UIKit
**UIKit framework**
- UI events
- View lifecycle

### com.apple.runningboard
**Process lifecycle**
- App states
- Memory management
- Background modes

### com.apple.frontboard
**Front-most app management**
- Scene management
- Window server

---

## Network

### com.apple.network
**Network framework**
- Connections
- Protocols

### com.apple.wifi
**WiFi**
- Association
- Scanning
- Power state

### com.apple.bluetooth
**Bluetooth**
- Device pairing
- Connections

### com.apple.corewlan
**CoreWLAN**
- WiFi management

### com.apple.apsd
**Apple Push Service**
- Push notifications
- Keep-alive

### com.apple.ids
**Identity Services**
- iMessage
- FaceTime
- iCloud accounts

---

## Storage & Data

### com.apple.coredata
**Core Data**
- Database operations
- Migrations

### com.apple.sqlite
**SQLite**
- Database queries
- Performance

### com.apple.cloudkit
**CloudKit**
- iCloud sync
- Records

### com.apple.icloud
**iCloud services**
- Sync state
- Account status

---

## System

### com.apple.kernel
**Kernel**
- System calls
- Resource limits

### com.apple.xpc
**XPC**
- Inter-process communication
- Service connections

### com.apple.launchd
**launchd**
- Service management
- Job scheduling

### com.apple.powerd
**Power management**
- Sleep/wake
- Thermal management

### com.apple.iokit
**I/O Kit**
- Hardware drivers
- Device state

---

## Media

### com.apple.avfoundation
**AVFoundation**
- Audio/video playback
- Capture

### com.apple.mediaserverd
**Media services**
- Audio routing
- Media sessions

### com.apple.coremedia
**Core Media**
- Media pipelines
- Buffers

---

## Communications

### com.apple.telephony
**Phone/Cellular**
- Calls
- SMS/MMS

### com.apple.messages
**Messages**
- iMessage
- SMS relay

### com.apple.facetime
**FaceTime**
- Video calls
- Audio calls

---

## Trial & Experiments

### com.apple.trial
**Trial system**
- Feature flags
- A/B experiments
- Rollouts

```bash
--predicate 'subsystem == "com.apple.trial"'
```

---

## High-Value Forensic Subsystems

| Priority | Subsystem | Use Case |
|----------|-----------|----------|
| **Critical** | com.apple.TCC | Permission analysis |
| **Critical** | com.apple.locationd | Location tracking |
| **Critical** | com.apple.springboard | User activity |
| **High** | com.apple.siri | Voice interactions |
| **High** | com.apple.biome | Behavior patterns |
| **High** | com.apple.intelligenceplatform | AI activity |
| **High** | com.apple.security | Security events |
| **Medium** | com.apple.runningboard | App lifecycle |
| **Medium** | com.apple.wifi | Network connections |
| **Medium** | com.apple.trial | Feature experiments |

---

## Predicate Patterns

### Single Subsystem
```bash
--predicate 'subsystem == "com.apple.springboard"'
```

### Subsystem Prefix
```bash
--predicate 'subsystem BEGINSWITH "com.apple.siri"'
```

### Multiple Subsystems
```bash
--predicate 'subsystem IN {"com.apple.TCC", "com.apple.security"}'
```

### Subsystem + Level
```bash
--predicate 'subsystem == "com.apple.security" AND messageType == error'
```

### Subsystem + Message
```bash
--predicate 'subsystem == "com.apple.siri" AND eventMessage CONTAINS "intent"'
```

---

## See Also

- [intelligence.md](intelligence.md) - Apple Intelligence subsystems
- [siri.md](siri.md) - Siri subsystems
- [../analysis/common-queries.md](../analysis/common-queries.md) - Query examples
