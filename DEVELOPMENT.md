# Development Guide - Car Swapper Plugin

Technical documentation for developers extending or modifying the plugin.

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│           CarSwapper (Main Plugin)              │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐  ┌──────────────┐            │
│  │   Hotkey     │  │    Config    │            │
│  │   Manager    │  │   Manager    │            │
│  └──────┬───────┘  └──────┬───────┘            │
│         │                 │                     │
│         └────────┬────────┘                     │
│                  │                              │
│            ┌─────▼─────┐                        │
│            │   Swap    │                        │
│            │  Engine   │                        │
│            └─────┬─────┘                        │
│                  │                              │
│         ┌────────▼─────────┐                    │
│         │   Car Database   │                    │
│         │  (Metadata,      │                    │
│         │   Physics, IDs)  │                    │
│         └──────────────────┘                    │
│                                                 │
└─────────────────────────────────────────────────┘
         │
         └──→ BakkesMod SDK (Game State)
```

## Class Hierarchy

### CarSwapper (Main)
- **Responsibility:** Orchestrates all components
- **Key Methods:** Initialize(), Update(), SwapCar(), ResetCar()
- **Dependencies:** CarDatabase, ConfigManager, HotkeyManager

### CarDatabase
- **Responsibility:** Stores and manages car metadata
- **Key Methods:** GetCarMetadata(), GetCarIdByName(), IsValidSwap()
- **Data:** Car IDs, names, manufacturers, hitbox dimensions

### ConfigManager
- **Responsibility:** Loads/saves configuration files
- **Key Methods:** LoadConfig(), SaveConfig(), GetPresets()
- **Data:** CarSwapPreset[], HotkeyConfig, PluginSettings

### HotkeyManager
- **Responsibility:** Monitors keyboard input and triggers callbacks
- **Key Methods:** Update(), RegisterPresetCallback(), CheckKeyState()
- **Input:** Windows Virtual Key codes

## Key Data Structures

### CarSwapPreset
```cpp
struct CarSwapPreset {
    std::string name;           // e.g., "octane_to_dominus"
    CarType sourceCarId;        // Source car (e.g., OCTANE)
    CarType targetCarId;        // Target car (e.g., DOMINUS)
    bool enabled;               // Whether preset is active
};
```

### SwapState
```cpp
struct SwapState {
    CarType originalCar;        // Original car before any swap
    CarType currentSwappedCar;  // Currently swapped car
    int activePresetIndex;      // Active preset index (-1 if none)
    bool isSwapped;             // Is a swap currently active?
    float lastSwapTime;         // Time of last swap (for cooldown)
};
```

### HotkeyConfig
```cpp
struct HotkeyConfig {
    int presetKeys[10];         // Shift+0 through Shift+9 (Virtual Key codes)
    int resetKey;               // Reset hotkey (default: 0)
    int reloadKey;              // Reload config hotkey (default: R)
    int modifierKey;            // Modifier key (default: Shift)
};
```

## Virtual Key Codes

Common Windows Virtual Key codes used in hotkey system:

```cpp
VK_SHIFT = 16              // Shift key
VK_CONTROL = 17            // Ctrl key
VK_MENU = 18               // Alt key
VK_0 through VK_9 = 48-57  // Number keys 0-9
VK_A through VK_Z = 65-90  // Letter keys A-Z
VK_F1 through VK_F12 = 112-123  // Function keys
```

For complete list: https://docs.microsoft.com/en-us/windows/win32/inputdev/virtual-key-codes

## Configuration File Format

**File:** `CarSwapper.cfg`

```ini
; Car Swapper Configuration
[CarSwaps]
Swap1=octane->dominus
Swap2=octane->fennec

[Hotkeys]
Preset0Key=48          ; 0 key
Preset1Key=49          ; 1 key
Preset2Key=50          ; 2 key
ResetKey=48            ; 0 key
ReloadKey=82           ; R key
ModifierKey=16         ; Shift key

[Settings]
EnableLogging=true
PreserveCameraSettings=false
EnablePhysicsSwap=true
```

## Integration with BakkesMod SDK

The plugin uses BakkesMod SDK for:

1. **Game State Access**
   ```cpp
   ServerWrapper game = gameWrapper->GetGameEventAsServer();
   PriWrapper playerCar = game.GetLocalCar();
   ```

2. **Vehicle Modification**
   - Changing vehicle model/mesh
   - Modifying physics properties
   - Updating camera offsets

3. **Event Hooks**
   - Game tick events
   - Car spawn events
   - Configuration reload events

### Implementation Points

Key methods that interface with BakkesMod:

```cpp
bool CarSwapper::ApplySwapToGameState(CarType targetCarId);
bool CarSwapper::RestoreOriginalCar();
```

These are currently placeholder implementations and need BakkesMod SDK integration.

## Extension Points

### Adding New Presets

In `ConfigManager`:
```cpp
CarSwapPreset newPreset;
newPreset.name = "custom_preset";
newPreset.sourceCarId = CarType::FENNEC;
newPreset.targetCarId = CarType::BATMOBILE;
newPreset.enabled = true;
m_configManager.AddPreset(newPreset);
```

### Custom Swap Callbacks

In your code:
```cpp
carSwapper.m_hotkeyManager.RegisterPresetCallback([](int index) {
    // Custom logic when preset hotkey pressed
    std::cout << "Preset " << index << " activated\n";
});
```

### Adding New Cars

In `CarDatabase::InitializeCarMetadata()`:
```cpp
m_cars.push_back({
    "New Car",           // Display name
    CarType::NEW_CAR,    // Car ID (add to enum)
    "Manufacturer",      // Manufacturer
    true,                // Is DLC?
    118.0f, 81.0f, 73.0f // Hitbox dimensions
});
```

Also add to `CarSwapDefines.h`:
```cpp
enum class CarType {
    // ... existing cars
    NEW_CAR = 999,  // New car ID
};

inline const std::map<std::string, CarType>& GetCarNameMap() {
    static const std::map<std::string, CarType> carMap = {
        // ... existing entries
        {"newcar", CarType::NEW_CAR},
    };
    return carMap;
}
```

## Compilation Flags

**Debug Build:**
```bash
cmake -G "Visual Studio 17 2022" -A x64 -DCMAKE_BUILD_TYPE=Debug ..
cmake --build . --config Debug
```

**Release Build:**
```bash
cmake --build . --config Release
```

## Code Style Guidelines

- **Naming:** CamelCase for classes, camelCase for variables
- **Comments:** Document public methods and complex logic
- **Error Handling:** Check return codes before proceeding
- **Memory:** Use smart pointers where appropriate
- **Constants:** Define in headers with `constexpr`

## Logging

Enable logging in config:
```ini
[Settings]
EnableLogging=true
```

Log messages via:
```cpp
carSwapper.Log("Your message here");
```

## Testing

### Manual Testing

1. Load plugin in BakkesMod
2. Enter Freeplay
3. Test hotkeys: Shift+1, Shift+2, Shift+0, Shift+R
4. Verify car models change correctly
5. Check `CarSwapper.log` for errors

### Debug Testing

Build with Debug configuration and use Visual Studio debugger:
```bash
cmake -G "Visual Studio 17 2022" -A x64 -DCMAKE_BUILD_TYPE=Debug ..
cmake --build . --config Debug
```

## Common Modifications

### Change Default Hotkeys

Edit `HotkeyConfig` constructor in `HotkeyManager.h`:
```cpp
HotkeyConfig() {
    presetKeys[0] = VK_F1;    // Change from 48 (0) to F1
    presetKeys[1] = VK_F2;    // Change from 49 (1) to F2
    // ... etc
    modifierKey = VK_CONTROL; // Change from 16 (Shift) to Ctrl
}
```

### Adjust Swap Cooldown

Edit `PluginSettings` in `CarSwapDefines.h`:
```cpp
struct PluginSettings {
    // ...
    float swapCooldown = 0.05f;  // Change from 0.1f to 0.05f
};
```

### Add Car Physics Adjustment

In `CarSwapper::ApplySwapToGameState()`:
```cpp
// Adjust physics after swap
if (m_configManager.GetSettings().enablePhysicsSwap) {
    float length, width, height;
    if (m_carDatabase.GetHitboxInfo(targetCarId, length, width, height)) {
        // Apply physics modifications
    }
}
```

## Performance Considerations

- **Memory:** Plugin uses ~2-5 MB
- **CPU:** Minimal impact (hotkey checking each frame)
- **Optimization:** Consider caching car metadata lookups

## Future Enhancements

Potential areas for extension:

1. **GUI Configuration Tool** - Graphical config editor
2. **Preset Profiles** - Save/load multiple configuration sets
3. **Advanced Physics** - Custom hitbox adjustments per swap
4. **Network Support** - Sync swaps with teammates in private matches
5. **Event System** - More granular callbacks and events
6. **Performance Metrics** - Track swap timing and success rates

## Debugging Tips

1. **Enable logging** in `CarSwapper.cfg`
2. **Check BakkesMod logs:** `C:\Program Files (x86)\BakkesMod\logs\`
3. **Use Visual Studio debugger** for step-through debugging
4. **Add debug breakpoints** in `HotkeyManager::Update()`
5. **Print swap state** in `CarSwapper::Update()`

---

**Questions? Check README.md or BUILD_GUIDE.md for more information.**
