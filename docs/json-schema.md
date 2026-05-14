# devices.json schema

Each entry in `devices.json` is keyed by hardware identifier and contains a flat object with 14 required fields.

## Example entry

```json
"iPhone17,3": {
    "identifier": "iPhone17,3",
    "screenSize": 6.1,
    "year": 2024,
    "marketingName": "iPhone 16",
    "specificModelRaw": "iPhone16",
    "genericModel": "iPhone16",
    "chip": "A18",
    "biometricSupport": "Face ID",
    "displayType": "OLED",
    "connectivity": "5G",
    "portType": "USB-C",
    "cameraMegapixels": 48,
    "hasUltraWide": true,
    "appleIntelligence": true
}
```

## Field reference

| Field | Type | Description | Notes |
|-------|------|-------------|-------|
| `identifier` | String | Hardware identifier | Must match the JSON key exactly |
| `screenSize` | Double | Screen diagonal in inches | `0` for simulators |
| `year` | Int | Year of release | |
| `marketingName` | String | Apple's official product name | e.g. `"iPhone 16 Pro Max"` |
| `specificModelRaw` | String | Unique model identifier | Must match a `Model` enum case |
| `genericModel` | String | Family grouping | Groups variants of the same generation |
| `chip` | String | SoC name | e.g. `"A18 Pro"`, `"M4"` |
| `biometricSupport` | String | Biometric capability | `""`, `"Touch ID"`, or `"Face ID"` |
| `displayType` | String | Display technology | See valid values below |
| `connectivity` | String | Best cellular/wireless standard | See valid values below |
| `portType` | String | Physical connector | `"30-pin"`, `"Lightning"`, or `"USB-C"` |
| `cameraMegapixels` | Double | Main camera resolution in MP | `0` for no camera. **Not exposed in `Device` struct.** |
| `hasUltraWide` | Bool | Has ultra-wide camera | |
| `appleIntelligence` | Bool | Supports Apple Intelligence | |

### Valid displayType values

- `"LCD"`
- `"OLED"`
- `"OLED ProMotion"`
- `"OLED (Ultra Retina XDR)"` — iPad Pro M4
- `"Mini-LED"` — iPad Pro M1/M2

### Valid connectivity values

- `"WiFi"` — Wi-Fi only
- `"WiFi+Cellular"` — cellular-capable
- `"2G"`, `"3G"`, `"4G"`, `"5G"` — generation-specific (iPhones and older iPods)
- `"simulator"` — simulator entries only

## Multiple identifiers per model

Many devices have multiple hardware identifiers (WiFi vs WiFi+Cellular, regional variants). These share the same `specificModelRaw` but differ in `identifier` and potentially `connectivity`.

Example — iPad Mini 7:
- `iPad16,1` → `"connectivity": "WiFi"`, `"specificModelRaw": "iPadMini7"`
- `iPad16,2` → `"connectivity": "WiFi+Cellular"`, `"specificModelRaw": "iPadMini7"`

The `Device.isiPad`, `Device.isiPhone`, `Device.isiPod` computed properties check the `identifier` field, not `specificModelRaw`.

## cameraMegapixels is intentionally absent from the Swift API

The field exists in all JSON entries but `Device` does not declare it as a property. Swift's synthesized `Codable` conformance ignores extra JSON keys, so decoding succeeds. The value is preserved in JSON for future use — when it's added to `Device`, no JSON migration is needed.

## Special entries

**Simulators** — three entries covering all simulator architectures:

| Key | Notes |
|-----|-------|
| `i386` | 32-bit simulator (legacy) |
| `x86_64` | Intel Mac simulator |
| `arm64` | Apple Silicon simulator |

All use `specificModelRaw: "simulator"` → `Model.simulator`.

## Naming conventions for specificModelRaw

PascalCase, no spaces or hyphens:

| Family | Pattern | Examples |
|--------|---------|---------|
| iPhone | `iPhone{generation}{variant}` | `iPhone16ProMax`, `iPhone16e`, `iPhoneAir1` |
| iPod | `iPodTouch{generation}` | `iPodTouch7` |
| iPad (standard) | `iPad{screenSize}Inch{generation}` | `iPad97Inch1`, `iPad109Inch10` |
| iPad Mini | `iPadMini{generation}` | `iPadMini7` |
| iPad Air | `iPadAir{screenSize}Inch{generation}` | `iPadAir11Inch7`, `iPadAir13Inch7` |
| iPad Pro | `iPadPro{screenSize}Inch{generation}` | `iPadPro11Inch7`, `iPadPro13Inch7` |
| Simulator | `simulator` | |

Screen size encoding (digits only, no decimal point):
`97` = 9.7", `102` = 10.2", `105` = 10.5", `109` = 10.9", `11` = 11", `129` = 12.9", `13` = 13"
