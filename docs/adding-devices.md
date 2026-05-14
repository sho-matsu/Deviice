# Adding new devices

When Apple announces new hardware, Deviice needs three files updated in sync.

## Before you start: find the hardware identifiers

Apple does not publish these. Sources:
- https://www.theiphonewiki.com/wiki/Models — most reliable, community-maintained
- https://everymac.com/systems/apple/
- ipsw.me — firmware-level identifiers

One physical device model typically has multiple hardware identifiers:
- iPhones: usually 2 (regional variants, or model numbers)
- iPads: 2–4 (WiFi, WiFi+Cellular, plus regional variants)
- iPod: 1–2

---

## Step 1: Add entries to devices.json

Add one JSON object per hardware identifier in `Sources/Deviice/devices.json`.

**Placement**: Within the file, maintain this order: simulators → iPod → iPhone → iPad.

Within each family, add new entries at the end of that family's block, in generation order.

**Example** — hypothetical iPhone 18:

```json
"iPhone19,3": {
    "identifier": "iPhone19,3",
    "screenSize": 6.3,
    "year": 2026,
    "marketingName": "iPhone 18",
    "specificModelRaw": "iPhone18",
    "genericModel": "iPhone18",
    "chip": "A20",
    "biometricSupport": "Face ID",
    "displayType": "OLED ProMotion",
    "connectivity": "5G",
    "portType": "USB-C",
    "cameraMegapixels": 48,
    "hasUltraWide": true,
    "appleIntelligence": true
},
```

For iPad models with WiFi and Cellular variants, add a separate entry per identifier:

```json
"iPad17,1": {
    "identifier": "iPad17,1",
    "specificModelRaw": "iPadMini8",
    "connectivity": "Wi-Fi",
    ...
},
"iPad17,2": {
    "identifier": "iPad17,2",
    "specificModelRaw": "iPadMini8",
    "connectivity": "WiFi+Cellular",
    ...
},
```

See `docs/json-schema.md` for the full field reference and valid values.

---

## Step 2: Add a Model enum case

Open `Sources/Deviice/Model.swift`. Add a new case in the correct MARK section.

The case name must **exactly** match the `specificModelRaw` value from the JSON.

```swift
// MARK: iPhone
// ... existing cases ...
case iPhone17ProMax
case iPhoneAir1
case iPhone18          // <-- new
```

Place it at the end of the relevant family group, in generation order.

If multiple identifiers share the same `specificModelRaw` (e.g. WiFi/Cellular variants), only one enum case is needed.

---

## Step 3: Run tests

```bash
swift test
```

The `validateModels` test iterates every JSON entry and checks that its `specificModelRaw` resolves to a valid `Model` case. If Step 2 was skipped, this test fails with a helpful print:

```
Identifier iPad17,1 is notMapped.
```

---

## Checklist

- [ ] All hardware identifiers added to devices.json (WiFi, Cellular, regional variants)
- [ ] `identifier` field inside the object matches the JSON key
- [ ] `specificModelRaw` follows naming convention (PascalCase, no spaces)
- [ ] New `Model` enum case added, name matches `specificModelRaw` exactly
- [ ] All 14 JSON fields present in each new entry
- [ ] `cameraMegapixels` populated (even though not yet exposed in the `Device` struct)
- [ ] `swift test` passes

---

## Common mistakes

**Forgetting Cellular variants** — iPads almost always have at least WiFi and WiFi+Cellular identifiers. Check all variants.

**Mismatched `identifier` vs JSON key** — the `identifier` field inside the object must be identical to the key. Copy-paste the key, don't retype it.

**Wrong `specificModelRaw` case name** — the enum case and the JSON string must be character-for-character identical. Swift enum matching is case-sensitive.

**Inconsistent connectivity values** — older entries use `"Wi-Fi"` (with hyphen), newer ones use `"WiFi"` (no hyphen). Match the convention used by nearby entries of the same era rather than introducing a third variant.

**Forgetting `cameraMegapixels`** — it's in the JSON schema but not in the Swift API. It still must be present in every entry. Use `0` for devices without a camera.
