# logs/ Directory Index

The `logs/` directory contains categorized subsystem logs, databases, and configuration dumps. This index covers all subdirectories found in iOS 18.1 sysdiagnoses.

## Directory Overview

| Directory | Contents | Forensic Value |
|-----------|----------|----------------|
| [Accessibility/](#accessibility) | TCC.db, accessibility settings | **Critical** |
| [powerlogs/](#powerlogs) | PowerLog database | **Critical** |
| [Trial/](#trial) | Feature flags, experiments | **High** |
| [GenerativeExperiences/](#generativeexperiences) | Apple Intelligence | **High** |
| [ModelCatalog/](#modelcatalog) | ML model state | **High** |
| [MobileInstallation/](#mobileinstallation) | App install logs | **High** |
| [Networking/](#networking) | Network diagnostics | **Medium** |
| [Bluetooth/](#bluetooth) | Bluetooth state | **Medium** |

---

## Privacy & Security

### Accessibility/
**Critical for forensics**

| File | Description |
|------|-------------|
| `TCC.db` | Privacy permission database |
| `*.plist` | Accessibility preferences |

```bash
sqlite3 logs/Accessibility/TCC.db "SELECT service, client, auth_value FROM access"
```

See: [../../privacy/tcc.md](../../privacy/tcc.md)

### AccessibilityPrefs/
Accessibility preference plists.

### profile_access/
Profile access logs for MDM/configuration profiles.

### ProtectedApps/
Protected apps configuration.

---

## AI / Machine Learning

### GenerativeExperiences/
**iOS 18+ only**

| File | Description |
|------|-------------|
| `GEAvailability.log` | Apple Intelligence availability |

```bash
cat logs/GenerativeExperiences/GEAvailability.log
```

See: [../../subsystems/intelligence.md](../../subsystems/intelligence.md)

### ModelCatalog/
ML model catalog state and configuration.

### ModelManager/
Model lifecycle management logs.

### ProactiveInputPredictions/
Predictive text/keyboard ML data.

---

## Power & Battery

### powerlogs/
**Critical - PowerLog database**

| File | Description |
|------|-------------|
| `powerlog_*.PLSQL` | SQLite telemetry database |

300+ tables with battery, app usage, location, and AI metrics.

See: [../../power/powerlog.md](../../power/powerlog.md)

### BatteryBDC/
Battery diagnostic data.

### BatteryHealth/
Battery health metrics and history.

### BatteryUIPlist/
Battery UI state (Settings app display).

### pmudiagnose/
Power Management Unit diagnostics.

---

## System Configuration

### Trial/
**Feature flag system**

| File | Description |
|------|-------------|
| `trial.log` | Main trial log |
| `trial-experiment-info.log` | Active experiments |
| `trial-namespace-compatibility-versions.log` | NCV values |
| `trial-rollout-info.log` | Rollout status |

```bash
cat logs/Trial/trial-namespace-compatibility-versions.log
```

See: [trial.md](trial.md)

### OSEligibility/
OS feature eligibility checks.

### SystemVersion/
System version information.

### MCState/
Mobile Configuration state (MDM).

### ManagedSettings/
Screen Time and parental controls.

---

## Network

### Networking/
Network diagnostic logs and state.

### NetworkRelay/
Network relay (iCloud Private Relay) logs.

### Bluetooth/
Bluetooth state and pairing info.

---

## Apps & Installation

### MobileInstallation/
**App installation logs**

Contains:
- App install/update history
- Provisioning profile info
- Code signing events

### MobileBackup/
Backup configuration and logs.

### MobileSoftwareUpdate/
Software update logs.

### MSU/
Mobile Software Update additional logs.

### OTAUpdateLogs/
Over-the-air update detailed logs.

### MobileActivation/
Device activation logs.

### MobileLockdown/
Lockdown daemon logs (USB pairing, etc.).

### itunesstored/
iTunes Store daemon logs.

### AppSupport/
App support files and logs.

---

## Media

### MobileSlideShow/
Photos app slideshow/memory data.

### CameraCapture/
Camera capture session logs.

---

## Communication

### Baseband/
Cellular baseband logs.

### atcrtcomm/
ATC (cellular) communication logs.

---

## Sensors

### SensorKit/
SensorKit framework logs (research studies).

---

## Miscellaneous

### ACLogs/
AutoCorrect/text input logs.

### AFK/
Activity/fitness kit logs.

### CalendarPreferences/
Calendar preferences.

### CloudSubscriptionFeatures/
iCloud subscription features state.

### FDR/
Factory Data Reset logs.

### IOSADiagnose/
iOS Accessibility diagnostics.

### keyboards/
Keyboard configuration and learned data.

### olddsc/
Old shared cache references.

### rmd/
Remote Management daemon logs.

### Sentry/
Sentry error tracking (if enabled).

### Splat/
Crash/splat diagnostics.

### swtransparency.log
Software transparency log (single file, not directory).

### UnifiedAsset/
Unified asset management logs.

### UserManagement/
User management logs.

---

## Quick Access Commands

### List All Logs Directories
```bash
ls -la logs/
```

### Find Databases
```bash
find logs/ -name "*.db" -o -name "*.sqlite" -o -name "*.PLSQL"
```

### Find Plists
```bash
find logs/ -name "*.plist"
```

### Check Directory Sizes
```bash
du -sh logs/*/
```

---

## See Also

- [../overview.md](../overview.md) - Top-level structure
- [../../databases/overview.md](../../databases/overview.md) - Database guide
- [../../privacy/tcc.md](../../privacy/tcc.md) - TCC analysis
