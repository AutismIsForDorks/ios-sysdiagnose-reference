# Network-Related Processes

Processes involved in networking, connectivity, and communications.

## WiFi

### wifid
**Path**: `/usr/sbin/wifid`

Core WiFi daemon:
- WiFi radio management
- Network association
- Scanning
- Power management

**High forensic value** - Network connectivity events.

```bash
--predicate 'process == "wifid"'
```

### wifianalyticsd
**Path**: `/usr/libexec/wifianalyticsd`

WiFi analytics:
- Connection quality metrics
- Network statistics

### wifivelocityd
**Path**: `/usr/libexec/wifivelocityd`

WiFi velocity/speed testing.

### wifip2pd
**Path**: `/usr/libexec/wifip2pd`

WiFi peer-to-peer (AirDrop, etc.).

### ThreeBarsXPCService
**Path**: WiFiPolicy framework XPC

WiFi signal strength analysis.

### WiFiCloudAssetsXPCService
**Path**: WiFiPolicy framework XPC

WiFi cloud configuration.

---

## Bluetooth

### bluetoothd
**Path**: `/usr/sbin/bluetoothd`

Core Bluetooth daemon:
- Bluetooth radio management
- Device pairing
- Connection management

```bash
--predicate 'process == "bluetoothd"'
```

### bluetoothuserd
**Path**: `/usr/libexec/bluetoothuserd`

Bluetooth user-space daemon:
- Bluetooth peripherals
- Audio devices

---

## Cellular

### CommCenter
Cellular communication center:
- Cellular radio
- Phone calls
- SMS/MMS

### WirelessRadioManagerd
**Path**: `/usr/sbin/WirelessRadioManagerd`

Radio management:
- Cellular/WiFi switching
- Radio state

### symptomsd
**Path**: `/usr/libexec/symptomsd`

Network diagnostics:
- Connection quality
- Network switching decisions

### symptomsd-diag
**Path**: `/usr/libexec/symptomsd-diag`

Extended network diagnostics.

---

## Core Networking

### configd
**Path**: `/usr/libexec/configd`

Network configuration daemon:
- Interface configuration
- DNS settings
- Network state

**Key process for network changes.**

```bash
--predicate 'process == "configd"'
```

### networkd
System network daemon:
- Network connections
- Socket management

### networkserviceproxy
**Path**: `/usr/libexec/networkserviceproxy`

Network service proxy:
- VPN
- Network extensions
- Proxy configuration

### mDNSResponder
Multicast DNS:
- Bonjour
- Local network discovery

---

## Apple Services

### apsd
**Path**: `/System/Library/PrivateFrameworks/ApplePushService.framework/apsd`

Apple Push Service daemon:
- Push notifications
- iCloud push
- App notifications

```bash
--predicate 'process == "apsd"'
```

### identityservicesd
**Path**: `/System/Library/PrivateFrameworks/IDS.framework/identityservicesd.app/identityservicesd`

Identity services:
- iMessage
- FaceTime
- iCloud account authentication

**Critical for communications analysis.**

### imagent
iMessage agent:
- iMessage delivery
- Message state

### facetimed
FaceTime daemon:
- FaceTime calls
- Audio/video sessions

### callservicesd
Call services:
- Phone calls
- VoIP

---

## Cloud & Sync

### cloudd
iCloud daemon:
- iCloud Drive
- Document sync

### bird
**Path**: iCloud Drive daemon

iCloud Drive file provider.

### nsurlsessiond
URL session daemon:
- Background downloads
- Network transfers

### sharingd
Sharing daemon:
- AirDrop
- Nearby sharing

---

## Network Security

### trustd
Trust evaluation:
- TLS certificates
- Trust decisions

### ocspd
OCSP daemon:
- Certificate revocation checks

### captiveagent
Captive portal agent:
- WiFi login pages
- Network authentication

---

## Process Discovery

### Find Network Processes

```bash
awk 'NR>1 {print $NF}' ps.txt | \
    grep -iE "wifi|network|bluetooth|cellular|radio|vpn|apsd|configd|ids" | sort | uniq
```

### Count Network Events

```bash
for proc in wifid bluetoothd configd apsd identityservicesd; do
    echo -n "$proc: "
    log show --archive system_logs.logarchive \
        --predicate "process == \"$proc\"" \
        --style json 2>/dev/null | grep -c '"timestamp"'
done
```

---

## Forensic Analysis

### Network Connectivity Timeline

```bash
# WiFi events
log show --archive system_logs.logarchive \
    --predicate 'process == "wifid" AND eventMessage CONTAINS "join"' \
    --style json

# Cross-reference with WiFi CSVs
ls WiFi/Entity_*_Join.csv.tgz
```

### Push Notification Activity

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "apsd"' \
    --style json | grep -c '"timestamp"'
```

### iMessage/FaceTime Activity

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "identityservicesd" OR process == "imagent"' \
    --style json
```

### Network State Changes

```bash
log show --archive system_logs.logarchive \
    --predicate 'process == "configd" AND eventMessage CONTAINS "state"' \
    --style json
```

---

## Related Artifacts

| Artifact | Location | Data |
|----------|----------|------|
| WiFi history | `WiFi/Entity_*_Join.csv` | Connection timeline |
| Known networks | `WiFi/com.apple.wifi.plist` | Saved networks |
| Bluetooth | `logs/Bluetooth/` | BT state |
| Network config | `Networking/` | Interface state |

---

## See Also

- [privacy.md](privacy.md) - Privacy processes
- [ai-ml.md](ai-ml.md) - AI processes
- [../index.md](../index.md) - All processes
- [../../network/wifi.md](../../network/wifi.md) - WiFi artifacts
