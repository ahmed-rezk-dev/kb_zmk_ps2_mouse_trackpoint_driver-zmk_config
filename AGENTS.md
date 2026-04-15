# AGENTS.md

ZMK keyboard firmware config for Corne with PS/2 TrackPoint support.

## Build

- CI uses `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.2`
- Local build: West handles everything via `config/west.yml` dependencies
- `build.yaml` defines the build matrix (board/shield combinations)

## Repository Structure

```
config/
  west.yml              # ZMK dependencies (fork, modules)
  corne_tp.keymap       # Main keymap (all layers)
  corne_tp.conf         # Build configuration
  include/
    mouse_tp.dtsi       # TrackPoint settings (sensitivity, inertia, etc.)

boards/shields/corne_tp/   # Corne shield with PS/2 TP support
  corne_tp.dtsi             # Matrix definition (3 rows x 6 cols per half)
  corne_tp_left.overlay     # Left half matrix config
  corne_tp_right.overlay    # Right half + PS/2 TrackPoint UART config
  Kconfig.defconfig         # Central side config
  Kconfig.shield            # Shield definition
```

## Corne Pinout

**Matrix pins:**
- Rows: D4, D5, D6, D7 (pro_micro pins 4, 5, 6, 7)
- Columns: D14, D15, D18, D19, D20, D21 (pro_micro pins 14, 15, 18, 19, 20, 21)

**PS/2 TrackPoint pins (default):**
- SCL: D16 (pro_micro pin 16)
- SDA: D10 (pro_micro pin 10)
- RST: D9 (pro_micro pin 9)

**Alternative pins (for keyboards without underglow):**
- SCL: D1, SDA: D0

## Keymap Editing

- **Feature flags**: `#define HAS_MOUSE_KEYS` and `#define HAS_MOUSE_TP` control conditional compilation
- Comment these out to build without mouse features
- TrackPoint settings can be configured at runtime via behaviors OR hardcoded in `config/include/mouse_tp.dtsi`

## ZMK Module Dependencies

Dependencies are in `config/west.yml`:
- ZMK fork: Uses `petejohanson/zmk@feat/pointers-move-scroll` (mouse PR)
- `kb_zmk_ps2_mouse_trackpoint_driver` from `badjeff` remote
- `zmk-helpers` from `urob` remote

To switch ZMK forks: Uncomment desired fork in `west.yml`, comment out others.

## Adding New Layers/Behaviors

1. Add layer in `config/corne_tp.keymap`
2. Update layer number defines (BASE, LOWER, MOUSE_KEYS, etc.)
3. Update `config/corne_tp.conf` if adding features

## PS/2 TrackPoint Configuration

- UART driver used (recommended for nRF52)
- BAUD rate: 14400 (standard for TrackPoints)
- Interrupt priorities pre-configured for BT compatibility
- TrackPoint activates MOUSE_TP layer automatically when moved
- Settings saved to flash after 60s (CONFIG_ZMK_SETTINGS_SAVE_DEBOUNCE)

## Feature Flags

```c
#define HAS_MOUSE_KEYS  // Enable mouse keys layer
#define HAS_MOUSE_TP    // Enable TrackPoint support
```

## Build Commands

```bash
# Local build (after west update)
west build -p -b nice_nano_v2 --shield corne_tp_left
west build -p -b nice_nano_v2 --shield corne_tp_right
```
