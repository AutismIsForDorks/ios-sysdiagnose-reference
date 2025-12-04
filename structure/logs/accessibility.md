# logs/Accessibility/ - TCC and Accessibility Data

Contains the TCC (Transparency, Consent, and Control) database and accessibility preferences.

## Directory Contents

| File | Description | Forensic Value |
|------|-------------|----------------|
| `TCC.db` | Privacy permission database | **Critical** |
| `*.plist` | Accessibility preferences | Low |

---

## TCC.db

The TCC database stores all privacy permission decisions.

### Location

```
logs/Accessibility/TCC.db
```

### Tables

| Table | Purpose |
|-------|---------|
| `access` | Permission grants/denials |
| `policies` | MDM policies |
| `active_policy` | Active policy links |
| `access_overrides` | System overrides |
| `expired` | Revoked permissions |
| `admin` | Admin settings |
| `integrity_flag` | Integrity markers |

### Quick Queries

```bash
# All permissions
sqlite3 logs/Accessibility/TCC.db "SELECT service, client, auth_value FROM access"

# Allowed permissions
sqlite3 logs/Accessibility/TCC.db "SELECT service, client FROM access WHERE auth_value = 2"

# Location access
sqlite3 logs/Accessibility/TCC.db "SELECT client FROM access WHERE service LIKE '%Location%' AND auth_value = 2"

# Recent changes
sqlite3 logs/Accessibility/TCC.db "
SELECT service, client, datetime(last_modified, 'unixepoch')
FROM access
ORDER BY last_modified DESC
LIMIT 20"
```

### Authorization Values

| Value | Meaning |
|-------|---------|
| 0 | Denied |
| 1 | Unknown |
| 2 | Allowed |
| 3 | Limited |

---

## Full Documentation

See: [../../privacy/tcc.md](../../privacy/tcc.md)

---

## See Also

- [index.md](index.md) - Logs directory index
- [../../privacy/tcc.md](../../privacy/tcc.md) - Full TCC guide
- [../../databases/schemas/tcc.md](../../databases/schemas/tcc.md) - Schema reference
