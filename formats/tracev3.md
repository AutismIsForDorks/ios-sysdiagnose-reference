# TraceV3 File Format

TraceV3 is Apple's binary format for unified logging data. This document describes the format structure for forensic analysis and tool development.

## Overview

TraceV3 files (`.tracev3`) contain compressed, structured log data from the iOS unified logging system. They are found in `system_logs.logarchive/`.

## File Locations

```
system_logs.logarchive/
├── logdata.LiveData.tracev3      # Current/live logs
├── Persist/
│   ├── 0000000000000001.tracev3  # Persistent logs
│   ├── 0000000000000002.tracev3
│   └── ...
├── Special/
│   └── *.tracev3                 # Long-retention logs
├── Signpost/
│   └── *.tracev3                 # Performance signposts
└── XX/                           # Hex directories (00-FF)
    └── *.tracev3                 # UUID-keyed chunks
```

---

## File Structure

### Header

TraceV3 files begin with a header containing:

| Offset | Size | Field |
|--------|------|-------|
| 0x00 | 4 | Magic (`0x0badf00d` or similar) |
| 0x04 | 4 | Version |
| 0x08 | 8 | Timestamp base |
| 0x10 | 4 | Chunk count |
| 0x14 | 4 | Flags |

### Chunk Structure

Data is organized in chunks:

```
+------------------+
| Chunk Header     |
+------------------+
| Compressed Data  |
+------------------+
| Chunk Footer     |
+------------------+
```

### Compression

TraceV3 uses LZ4 or LZFSE compression:

| Compression | Identifier |
|-------------|------------|
| LZ4 | `bv4-` |
| LZFSE | `bvx-` |
| Uncompressed | `bv0-` |

---

## Log Entry Structure

After decompression, entries have:

| Field | Type | Description |
|-------|------|-------------|
| Timestamp | uint64 | Continuous time (nanoseconds) |
| Thread ID | uint64 | Thread identifier |
| Process ID | uint32 | Process identifier |
| Subsystem | string | Logging subsystem |
| Category | string | Log category |
| Message | string | Log message (format + args) |
| Level | uint8 | Log level |

### Log Levels

| Value | Level |
|-------|-------|
| 0x00 | Default |
| 0x01 | Info |
| 0x02 | Debug |
| 0x10 | Error |
| 0x11 | Fault |

---

## String Tables

TraceV3 uses string tables for efficiency:

- Subsystem names
- Category names
- Format strings
- Process paths

Strings are referenced by index, stored separately.

---

## UUID Mapping

The `dsc/` directory contains UUID-to-path mappings:

```
dsc/
└── uuidtext/
    └── XX/
        └── XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

Used to resolve image UUIDs to framework names.

---

## Reading TraceV3

### Using log show (Recommended)

```bash
# Apple's official tool
log show --archive system_logs.logarchive
```

### Using Python

```python
# Basic structure inspection
import struct

with open('file.tracev3', 'rb') as f:
    magic = struct.unpack('<I', f.read(4))[0]
    version = struct.unpack('<I', f.read(4))[0]
    print(f"Magic: {hex(magic)}, Version: {version}")
```

### Using ipsw

```bash
# Third-party tool
ipsw log show system_logs.logarchive
```

---

## Catalog Files

The logarchive may contain catalog files:

| File | Purpose |
|------|---------|
| `Info.plist` | Archive metadata |
| `*.catalog` | Message format catalogs |
| `timesync/` | Time synchronization |

### Info.plist Fields

```xml
ArchiveIdentifier: UUID
OSArchiveVersion: 5
EndTimeRef:
  ContinuousTime: nanoseconds
  WallTime: nanoseconds since 1970
  UUID: boot session
```

---

## Time Handling

### Continuous Time

Logs use "continuous time" - nanoseconds since boot:

```
timestamp = continuous_time_base + entry_offset
```

### Wall Clock Conversion

Wall time mapping in `timesync/`:

```
wall_time = continuous_time + time_offset
```

### Boot Sessions

Each boot has a UUID. Timestamps are relative to boot.

---

## Privacy Markers

Sensitive data is marked with privacy qualifiers:

| Marker | Meaning |
|--------|---------|
| `<private>` | Redacted in production |
| `<mask.hash>` | Hashed value |
| `<sensitive>` | Sensitive data marker |

In debug builds or with profiles, these may be visible.

---

## Chunk Types

| Type | Contents |
|------|----------|
| `Persist` | Standard persistent logs |
| `Special` | Long-retention logs |
| `Signpost` | Performance intervals |
| `HighVolume` | High-frequency events |
| `Live` | Current session |

---

## Forensic Considerations

### Data Recovery

Even "deleted" logs may persist in:
- Unallocated chunk space
- Failed compression blocks
- Backup residue

### Integrity

No built-in integrity verification. Compare:
- File sizes across archives
- Entry counts
- Time ranges

### Timestamps

Continuous time is more reliable than wall time:
- Not affected by time zone changes
- Not affected by user time manipulation
- Consistent within boot session

---

## Tools

### Apple

| Tool | Platform | Notes |
|------|----------|-------|
| `log` | macOS | Official, full support |
| Console.app | macOS | GUI viewer |

### Third-Party

| Tool | Notes |
|------|-------|
| [ipsw](https://github.com/blacktop/ipsw) | Cross-platform |
| [UnifiedLogReader](https://github.com/ydkhatri/UnifiedLogReader) | Python library |
| [macos-UnifiedLogs](https://github.com/mandiant/macos-UnifiedLogs) | Rust library |

---

## Limitations

### No Direct Parsing

TraceV3 is not meant for direct parsing:
- Complex compression
- Internal references
- Version changes

Use `log show` when possible.

### Version Differences

Format changes between iOS versions:
- iOS 14: OSArchiveVersion 4
- iOS 15+: OSArchiveVersion 5
- Internal structure varies

### Incomplete Documentation

Apple has not published official format documentation.

---

## See Also

- [../structure/system_logs.md](../structure/system_logs.md) - Log archive structure
- [ips.md](ips.md) - Crash report format
- [../analysis/common-queries.md](../analysis/common-queries.md) - Log queries
