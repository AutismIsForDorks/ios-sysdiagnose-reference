# WiFi/ Directory - Network Artifacts

The `WiFi/` directory contains comprehensive WiFi diagnostics including connection history, known networks, scan results, and network policies.

## Directory Contents

### Configuration Files

| File | Description | Forensic Value |
|------|-------------|----------------|
| `com.apple.wifi.plist` | Current WiFi configuration | **High** - Current network state |
| `com.apple.wifi-private-mac-networks.plist` | Private MAC address settings | **Medium** |
| `LEGACY_com.apple.wifi-networks.plist` | Legacy network list | **Medium** |

### Diagnostic Logs

| File | Description |
|------|-------------|
| `debug-log.txt` | WiFi debug output |
| `wifi_logarchive.log` | WiFi-specific logs |
| `diagnostics-configuration.txt` | WiFi config diagnostics |
| `diagnostics-connectivity.txt` | Connectivity test results |
| `diagnostics-environment.txt` | RF environment data |

### Status Files

| File | Description |
|------|-------------|
| `wifi_status.txt` | Current WiFi status |
| `wifi_scan.txt` | Recent scan results |
| `wifi_scan_cache.txt` | Cached scan data |
| `awdl_status.txt` | Apple Wireless Direct Link status |
| `network_status.txt` | Overall network status |
| `bluetooth_status.txt` | Bluetooth coexistence |

### Network Tools Output

| File | Description |
|------|-------------|
| `ifconfig.txt` | Interface configuration |
| `netstat-PRE.txt` | Network statistics (pre-diag) |
| `netstat-POST.txt` | Network statistics (post-diag) |
| `arp.txt` | ARP table |
| `3bars.txt` | Signal strength info |

### Data Path Info

| File | Description |
|------|-------------|
| `wifi_datapath-PRE.txt` | Data path state (pre) |
| `wifi_datapath-POST.txt` | Data path state (post) |

## CSV Entity Files (Compressed)

These `.tgz` files contain rich historical data:

| File Pattern | Contents |
|--------------|----------|
| `Entity_*_Join.csv` | **Network join history** - timestamps, SSIDs |
| `Entity_*_Leave.csv` | Network disconnection events |
| `Entity_*_Network.csv` | Known network database |
| `Entity_*_BSS.csv` | Basic Service Set info (access points) |
| `Entity_*_Scan.csv` | WiFi scan history |
| `Entity_*_Roam.csv` | Roaming events between APs |
| `Entity_*_Event.csv` | General WiFi events |
| `Entity_*_Fault.csv` | WiFi faults/errors |
| `Entity_*_Recovery.csv` | Recovery attempts |
| `Entity_*_LinkTest.csv` | Link quality tests |
| `Entity_*_LAN.csv` | LAN connectivity |
| `Entity_*_DiagnosticState.csv` | Diagnostic state snapshots |
| `Entity_*_MetricEntry.csv` | Performance metrics |
| `Entity_*_UsageMonthly.csv` | Monthly usage statistics |
| `Entity_*_UsageWeekly.csv` | Weekly usage statistics |
| `Entity_*_WiFiStat.csv` | WiFi statistics |
| `Entity_*_Policies*.csv` | Network policies |

## Extracting CSV Data

```bash
# Extract a specific CSV
tar -xzf "Entity_2025-11-22_19:36:51.452_Join.csv.tgz"

# Extract all CSVs to temp directory
mkdir /tmp/wifi_csv
for f in Entity_*.tgz; do
    tar -xzf "$f" -C /tmp/wifi_csv
done
```

## Join History Analysis

The `Join.csv` file is crucial for timeline reconstruction:

### CSV Columns (typical)

| Column | Description |
|--------|-------------|
| `timestamp` | Connection time (Unix epoch) |
| `networkName` | SSID (may be hashed/redacted) |
| `securityType` | WPA2, WPA3, Open, etc. |
| `rssi` | Signal strength at join |
| `channel` | WiFi channel |
| `bssid` | Access point MAC (may be redacted) |
| `reason` | Join reason |

### Analysis Commands

```bash
# Count connections per network
awk -F',' 'NR>1 {print $2}' Join.csv | sort | uniq -c | sort -rn

# Timeline of connections
awk -F',' 'NR>1 {print $1, $2}' Join.csv | \
    while read ts net; do
        date -r "$ts" "+%Y-%m-%d %H:%M - $net"
    done

# Find connections in time range
awk -F',' -v start=1732000000 -v end=1733000000 \
    'NR>1 && $1>=start && $1<=end {print}' Join.csv
```

## Known Networks (Plist)

Parse `com.apple.wifi.plist`:

```bash
plutil -p com.apple.wifi.plist
```

Contains:
- Known network SSIDs
- Auto-join preferences
- Private MAC settings per network
- Last connected timestamps
- Security configurations

## AWDL Status

Apple Wireless Direct Link is used for:
- AirDrop
- AirPlay
- Sidecar

`awdl_status.txt` shows:
- AWDL interface state
- Peer devices discovered
- Active sessions

## Forensic Applications

### Timeline Construction

1. Extract Join/Leave CSVs
2. Correlate with unified logs
3. Map to location data (if available)

### Network Profiling

- Identify frequently visited locations
- Detect unusual network connections
- Find enterprise vs personal networks

### Security Analysis

- Check for connections to open networks
- Identify potential rogue APs
- Review security type downgrades

## Privacy Considerations

- SSIDs may reveal location patterns
- BSSIDs can be geolocated
- Connection timestamps establish presence
- Private MAC reduces tracking but logs contain real MACs

## Delta Analysis

Compare WiFi artifacts across sysdiagnoses:

```bash
# Compare known networks
diff <(plutil -p archive1/WiFi/com.apple.wifi.plist | grep -i ssid) \
     <(plutil -p archive2/WiFi/com.apple.wifi.plist | grep -i ssid)

# Compare join counts
for archive in archive1 archive2; do
    echo "=== $archive ==="
    tar -xzf "$archive/WiFi/Entity_*_Join.csv.tgz" -O | wc -l
done
```

## Related Unified Logs

```bash
# WiFi subsystem logs
log show --archive system_logs.logarchive \
    --predicate 'subsystem == "com.apple.wifi"'

# Network extension logs
log show --archive system_logs.logarchive \
    --predicate 'process == "wifid" OR process == "airportd"'
```

## See Also

- [bluetooth.md](bluetooth.md) - Bluetooth artifacts
- [../privacy/location.md](../privacy/location.md) - Location correlation
- [../analysis/timeline-construction.md](../analysis/timeline-construction.md) - Timeline building
