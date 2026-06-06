# Complete Change History: pkg/memory/offset.go

## Overview
This file documents all structural and data changes to `pkg/memory/offset.go` throughout the repository history. The file primarily contains **hardcoded memory offsets** that must be updated when Diablo 2 Resurrected patches are released.

---

## Major Structural Evolution

### Phase 1: Initial Simple Offset Structure (Early Development)
**Commit:** `d2da4b19` (Early in history)
- **Struct Fields:** 10 basic fields
  - `GameData, UnitTable, UI, Hover, Expansion`
  - `RosterOffset, PanelManagerContainerOffset, WidgetStatesOffset`
  - `WaypointsOffset, FPS`
- **Function:** `calculateOffsets(process Process)` - simpler parameter passing
- **Offset Method:** Used hardcoded addresses in return struct (no dynamic lookup)

**Code Example:**
```go
type Offset struct {
    GameData                    uintptr
    UnitTable                   uintptr
    UI                          uintptr
    Hover                       uintptr
    Expansion                   uintptr
    RosterOffset                uintptr
    PanelManagerContainerOffset uintptr
    WidgetStatesOffset          uintptr
    WaypointsOffset             uintptr
    FPS                         uintptr
}
```

---

### Phase 2: Major Expansion to Dynamic Pattern Matching (Current)
**Commits:** `3f559332` onwards (Recent)
- **Struct Fields Increased:** From 10 to 20 fields
  - Added: `KeyBindingsOffset, KeyBindingsSkillsOffset, QuestInfo, TZ, Quests, Ping, LegacyGraphics, CharData`
  - Renamed: `WaypointsOffset` → `WaypointTableOffset`
  - Added: `SelectedCharName, LastGameName, LastGamePassword`
- **Function Signature Changed:** `calculateOffsets(process *Process)` (now pointer)
- **Offset Calculation:** Shifted to dynamic pattern matching using signature scanning
  - Uses `FindPattern()` and `FindPatternByOperand()` methods
  - Reads relative offsets from memory instead of hardcoding absolute addresses

**New Code Structure:**
```go
type Offset struct {
    GameData                    uintptr
    UnitTable                   uintptr
    UI                          uintptr
    Hover                       uintptr
    Expansion                   uintptr
    RosterOffset                uintptr
    PanelManagerContainerOffset uintptr
    WidgetStatesOffset          uintptr
    WaypointTableOffset         uintptr    // Renamed from WaypointsOffset
    FPS                         uintptr
    KeyBindingsOffset           uintptr    // NEW
    KeyBindingsSkillsOffset     uintptr    // NEW
    QuestInfo                   uintptr    // NEW
    TZ                          uintptr    // NEW
    Quests                      uintptr    // NEW
    Ping                        uintptr    // NEW
    LegacyGraphics              uintptr    // NEW
    CharData                    uintptr    // NEW
    SelectedCharName            uintptr    // NEW
    LastGameName                uintptr    // NEW
    LastGamePassword            uintptr    // NEW
}
```

---

## Offset Updates Timeline

### Recent Offset Updates (Last 6 Months)

| Date | Commit | Change Description |
|------|--------|-------------------|
| **Mar 17, 2026** | `dba427c` | **Update offsets** (most recent) |
| **Feb 19, 2026** | `b3f2b06` | update offsets |
| **Jan 23, 2026** | `ba3d781` | update remaining offsets |
| **Dec 11, 2025** | `3492085` | update tz and game name/password offsets |
| **Dec 11, 2025** | `630643d` | update GetSelectedCharacterName offset |
| **Dec 11, 2025** | `4a55faa` | update offset for reading key bindings |
| **Dec 17, 2025** | `be202aa` | fix offsets |
| **Nov 5, 2025** | `b644d79` | fix offsets after d2r update |

### Pattern: Response to Game Patches
The frequent offset updates follow this pattern:
1. Diablo 2 Resurrected receives a patch
2. Game memory layout shifts (new code = new addresses)
3. Pattern scanning detects new memory locations
4. Offsets are updated in this file
5. Tools relying on memory reading continue functioning

---

## Key Offset Calculation Methods

### Method 1: Direct Signature Scanning (Most Offsets)
```go
pattern := process.FindPattern(memory, "byte_pattern", "mask_pattern")
offsetValue := process.ReadUInt(pattern+offset, Uint32)
result := pattern - process.moduleBaseAddressPtr + 7 + uintptr(offsetValue)
```

**Used for:**
- `UnitTable, UI, Hover, Expansion, RosterOffset`
- `PanelManagerContainer, WidgetStates, WaypointTable, FPS`
- `QuestInfo, Ping, LegacyGraphics, CharData`

### Method 2: Operand-Based Scanning
```go
pattern := process.FindPatternByOperand(memory, "pattern", "mask")
panelManagerContainerOffset := (pattern - process.moduleBaseAddressPtr)
```

**Used for:**
- `PanelManagerContainerOffset`

### Method 3: Hardcoded (Legacy/Special Cases)
```go
tzOffset := uintptr(0x29B2DF0)  // Terror Zones offset
```

**Used for:**
- `TZ (Terror Zones)` - appears to be a stable address

### Method 4: Complex Relative Offset Reading
```go
bytes := process.ReadBytesFromMemory(pattern+3, 4)
relativeOffset := int32(binary.LittleEndian.Uint32(bytes))
resultOffset := pattern - process.moduleBaseAddressPtr + 7 + uintptr(relativeOffset)
```

**Used for:**
- `KeyBindings, KeyBindingsSkills, CharData, SelectedCharName`

---

## Specific Offset Values Changes

### Terror Zones (TZ) Offset
- **Dec 11, 2025** (`3492085`): Updated to `0x25B4990`
- **Previous:** `0x29B2DF0` (much different address)

### Game Name / Password Offsets  
- **Nov 25, 2025** (`4cf61a34`): 
  - `LastGameName`: `0x25FD2F0`
  - `LastGamePassword`: `0x25FD348`

### Key Bindings Offset
- **Dec 11, 2025** (`4a55faa`): Updated using pattern scanning
- Changed from: `0x19D5594` (old hardcoded)

### Selected Character Name Offset
- **Dec 11, 2025** (`630643d`): Updated using pattern scanning
- Changed from: `0x1D53195` (old hardcoded)

---

## Impact of Changes

### What Changes
✅ **Memory addresses only** - The structure remains compatible
✅ **Pattern signatures** - Detection methods may evolve
✅ **Offset calculation logic** - May improve over time

### What Doesn't Change
❌ **Struct field names** - API remains stable
❌ **Function signatures** - Code using this file requires no changes
❌ **Calculation methodology** - Pattern-based approach is consistent

---

## Why These Updates Are Necessary

1. **ASLR (Address Space Layout Randomization)**: Not directly applicable, but code changes shift layouts
2. **Game Patches**: D2R receives regular patches that:
   - Add new code sections
   - Reorganize existing code
   - Shift memory layout
3. **Signature Scanning**: Allows automatic discovery of new addresses based on bytecode patterns
4. **Maintenance**: Community contributors identify and submit new offsets after each patch

---

## How to Identify When Updates Are Needed

Monitor for:
- D2R patch announcements
- Function failures related to memory reading
- Community reports of broken tools/bots
- GitHub issues in this repository

---

## References

- **Repository:** https://github.com/andrzejzmudaa/d2go
- **File:** `/pkg/memory/offset.go`
- **Related Files:**
  - `pkg/memory/process.go` - Memory reading implementation
  - `pkg/memory/address.go` - Address utilities
  - Any files using `CalculateOffsets()` function

---

## Summary

The `offset.go` file is the **heart of memory reading** in this codebase. It transforms from simple hardcoded addresses to sophisticated **pattern-based dynamic offset discovery**. This evolution allows tools to remain functional across game patches without constant manual updates—a critical requirement for tools reading live game memory.
