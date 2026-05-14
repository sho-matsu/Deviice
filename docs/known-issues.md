# Known issues and technical debt

## cameraMegapixels not exposed in public API

`cameraMegapixels` is present in every JSON entry but not declared as a property on `Device`. The synthesized `Codable` conformance silently ignores it.

**Impact**: None currently. Adding it would be a minor, additive API change. No JSON migration needed.

---

## Device.current and Device.init() have different nil semantics

`Device.current` returns `Device?` — it returns `nil` if the hardware identifier is not in the JSON.

`Device.init(identifier:)` returns a non-optional `Device` — unknown identifiers produce a device with fallback values (`0`, `"-"`, `false`).

**Impact**: Medium. The inconsistency is surprising. Callers using `Device()` may not realize they have an unmapped device unless they check `isNotMapped`.

---

## No CHANGELOG

Version history is only available through git tags and commit messages. 43 tags exist from `0.20.00` to `3.0`.

**Fix**: Add a `CHANGELOG.md` for the next release.
