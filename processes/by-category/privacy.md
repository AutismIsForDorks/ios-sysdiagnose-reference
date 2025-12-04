# Privacy-Related Processes

Processes involved in privacy enforcement, data protection, and sensitive data handling.

## Permission & Consent

### tccd
**Path**: `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd`

TCC daemon - enforces all privacy permissions:
- Camera, microphone access
- Location, contacts, photos
- All permission prompts

**Log predicate**:
```bash
--predicate 'process == "tccd"'
```

**Critical forensic value** - All permission decisions flow through tccd.

### privacyaccountingd
**Path**: `/System/Library/PrivateFrameworks/PrivacyAccounting.framework/Versions/A/Resources/privacyaccountingd`

Privacy accounting daemon:
- Tracks data access by apps
- Generates App Privacy Reports
- Logs sensitive data usage

### adprivacyd
**Path**: `/usr/libexec/adprivacyd`

Advertising privacy daemon:
- App Tracking Transparency
- Ad attribution
- Privacy-preserving ad measurement

### webprivacyd
**Path**: `/System/Library/PrivateFrameworks/WebPrivacy.framework/webprivacyd`

Web privacy daemon:
- Intelligent Tracking Prevention
- Cross-site tracking protection
- Safari privacy features

---

## Location

### locationd
**Path**: `/usr/libexec/locationd`

Core location daemon:
- GPS/GNSS management
- WiFi-based location
- Cell tower location
- Authorization enforcement

**High forensic value** - Central to location tracking.

```bash
--predicate 'process == "locationd"'
```

### routined
**Path**: `/usr/libexec/routined`

Significant locations:
- Learns frequently visited places
- Predicts user location
- Powers location suggestions

**Contains sensitive location history.**

### geod
**Path**: (System framework)

Geocoding daemon:
- Address ↔ coordinate conversion
- Map data queries

---

## Behavioral Data

### biomed
**Path**: `/System/Library/PrivateFrameworks/BiomeStreams.framework/Support/biomed`

Biome daemon - behavioral data:
- App usage patterns
- Device interactions
- User routines

**Critical for forensics** - Comprehensive behavior tracking.

```bash
--predicate 'process == "biomed"'
```

### biomesyncd
**Path**: `/usr/libexec/biomesyncd`

Biome sync daemon:
- iCloud sync for Biome data
- Cross-device behavior sync

### coreduetd
**Path**: `/usr/libexec/coreduetd`

Core Duet daemon:
- Background task scheduling
- Activity prediction
- Power-aware execution

### duetexpertd
**Path**: `/usr/libexec/duetexpertd`

Duet expert daemon:
- ML-based predictions
- Usage pattern analysis

### contextstored
**Path**: `/System/Library/PrivateFrameworks/CoreDuetContext.framework/Resources/contextstored`

Context storage:
- User context tracking
- Situational awareness data

### parsecd
**Path**: `/System/Library/PrivateFrameworks/CoreParsec.framework/parsecd`

Siri personalization:
- Search ranking
- Contact importance
- App suggestions

**Contains personalization data.**

---

## Health & Fitness

### healthd
**Path**: `/System/Library/Frameworks/HealthKit.framework/healthd`

HealthKit daemon:
- Health data storage
- Workout data
- Medical records

**Highly sensitive personal data.**

### healthrecordsd
Health records daemon:
- Clinical records
- Lab results
- Medications

### fitnessintelligenced
**Path**: `/System/Library/PrivateFrameworks/FitnessIntelligenceDaemonCore.framework/fitnessintelligenced`

Fitness intelligence:
- Workout detection
- Activity classification
- Fitness predictions

---

## Security

### securityd
**Path**: `/usr/libexec/securityd`

Security daemon:
- Keychain operations
- Code signing verification
- Certificate management

### keybagd
**Path**: `/usr/libexec/keybagd`

Keybag daemon:
- Encryption key management
- Data protection classes
- Secure enclave interface

### trustd
**Path**: (System daemon)

Trust daemon:
- Certificate trust evaluation
- Trust store management

### amfid
**Path**: `/usr/libexec/amfid`

Apple Mobile File Integrity:
- Code signature verification
- Entitlement enforcement

---

## Identity & Authentication

### identityservicesd
**Path**: `/System/Library/PrivateFrameworks/IDS.framework/identityservicesd.app/identityservicesd`

Identity services:
- iMessage/FaceTime identity
- Apple ID authentication
- Device-to-device encryption

### biometrickitd
**Path**: `/usr/libexec/biometrickitd`

Biometric authentication:
- Face ID
- Touch ID
- Biometric data (never leaves Secure Enclave)

### fairplaydeviceidentityd
**Path**: `/usr/libexec/fairplaydeviceidentityd`

FairPlay device identity:
- DRM identity
- Device attestation

---

## Process Discovery

### Find Privacy Processes

```bash
awk 'NR>1 {print $NF}' ps.txt | \
    grep -iE "tcc|location|biome|parsec|health|security|identity|privacy" | sort | uniq
```

### Count Privacy Process Events

```bash
for proc in tccd locationd biomed parsecd healthd; do
    echo -n "$proc: "
    log show --archive system_logs.logarchive \
        --predicate "process == \"$proc\"" \
        --style json 2>/dev/null | grep -c '"timestamp"'
done
```

---

## Forensic Analysis

### Permission Timeline

Cross-reference tccd logs with TCC.db:

```bash
# tccd decisions
log show --archive system_logs.logarchive \
    --predicate 'process == "tccd" AND eventMessage CONTAINS "authorization"' \
    --style json

# Compare with TCC.db
sqlite3 logs/Accessibility/TCC.db "SELECT * FROM access ORDER BY last_modified DESC LIMIT 20"
```

### Location Activity

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "locationd"' \
    --style json | grep -c '"timestamp"'
```

### Behavioral Data Access

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "biomed" OR process == "parsecd"' \
    --style json
```

---

## See Also

- [ai-ml.md](ai-ml.md) - AI processes (also privacy-relevant)
- [network.md](network.md) - Network processes
- [../index.md](../index.md) - All processes
- [../../privacy/tcc.md](../../privacy/tcc.md) - TCC analysis
- [../../privacy/biome.md](../../privacy/biome.md) - Biome data
