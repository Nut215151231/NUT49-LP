# NUT49-LP

ZMK firmware for a custom split ergonomic keyboard with PMW3610 trackball and rotary encoders.

## Features

- **Split design** — Left and right halves connected via Bluetooth
- **PMW3610 trackball** on right side (1400 CPI)
- **Rotary encoders** — Left: 2 encoders (horizontal/vertical scroll), Right: 1 encoder (volume)
- **82 keys** total (6 rows × 7 columns per side, row 3 has 12 keys)
- **ZMK Studio support** — Edit keymap in browser without reflashing
- **Auto Mouse Layer** — Mouse layer activates automatically when trackball is moved
- **Snipe mode** — Low-speed trackball mode for precision
- **nice!nano v2** microcontrollers

## Hardware

### Components

- 2× nice!nano v2
- 1× PMW3610 trackball sensor
- 3× EC11 rotary encoders (2 left, 1 right)
- 82× switches + diodes
- 2× LiPo batteries

### Pin Assignments

| Function | Left GPIO | Right GPIO |
|---|---|---|
| Col 0–6 | P1.13, P1.11, P0.10, P0.09, P1.06, P1.04, P0.11 | P1.00, P0.11, P1.04, P1.06, P0.09, P0.10, P1.11 |
| Row 0–5 | P0.06, P0.08, P0.31, P0.29, P0.17, P0.20 | P0.06, P0.08, P0.17, P0.20, P0.31, P0.29 |
| Encoder 1 A/B | P0.22 / P0.02 | — |
| Encoder 2 A/B | P1.15 / P0.24 | — |
| Encoder R A/B | — | P0.02 / P0.22 |
| SPI SCK/MOSI/MISO | — | P1.01 / P1.02 / P1.02 |
| Trackball CS | — | P1.07 |
| Trackball IRQ | — | P0.24 |

## Repository Structure

```
config/
├── west.yml                           # ZMK + PMW3610 driver modules
├── build.yaml                         # Build targets (left, right, settings_reset)
└── boards/shields/NUT49_LP/
    ├── NUT49_LP.dtsi                  # Common DTS (matrix transform, encoders)
    ├── NUT49_LP_layouts.dtsi          # Physical layout definition
    ├── NUT49_LP_physical_layout.dtsi  # Key coordinates for ZMK Studio
    ├── NUT49_LP.keymap                # Keymap definition
    ├── NUT49_LP.json                  # KLE layout JSON
    ├── NUT49_LP_left.overlay          # Left hardware (kscan, encoders)
    ├── NUT49_LP_left.conf             # Left config (PERIPHERAL)
    ├── NUT49_LP_right.overlay         # Right hardware (kscan, trackball, encoder)
    ├── NUT49_LP_right.conf            # Right config (CENTRAL, ZMK Studio)
    ├── Kconfig.shield
    └── Kconfig.defconfig
```

## Keymap Layers

| Layer | Activation | Description |
|---|---|---|
| 0 BASE | Default | Main typing layer |
| 1 LOWER | Hold Lower key | Numbers, symbols, F keys |
| 2 RAISE | Hold Raise key | Navigation, symbols |
| 3 MOUSE | Auto (trackball) | Mouse buttons, snipe |
| 4 FUNC | LOWER + RAISE | BT, USB, system |
| 5 SNIPE | Hold M in MOUSE | Low-speed trackball |

## Encoder Bindings

| Encoder | Default | LOWER layer |
|---|---|---|
| Left 1 (horizontal) | Horizontal scroll | Horizontal scroll |
| Left 2 (vertical) | Vertical scroll | Page Up/Down |
| Right | Volume Up/Down | Volume Up/Down |

## Trackball

- Sensor: PMW3610
- Driver: [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)
- CPI: 1400
- Auto Mouse Layer: activates on movement, exits after 1 second idle
- AML misfire prevention: `require-prior-idle-ms = <200>` (no AML within 200ms of keypress)
- Snipe mode: hold M key in MOUSE layer for reduced speed

To adjust CPI, edit `NUT49_LP_right.overlay`:

```c
trackball: trackball@0 {
    cpi = <1400>;
};
```

## Build

### GitHub Actions (recommended)

Push to `main` branch to trigger automatic build. Download firmware from the Actions artifacts.

Three artifacts are produced:
- `NUT49_LP_left-nice_nano_v2-zmk.uf2`
- `NUT49_LP_right-nice_nano_v2-zmk.uf2`
- `settings_reset-nice_nano_v2-zmk.uf2`

### Local build

```bash
west init -l config/
west update
west build -d build/right -b nice_nano_v2 -- -DSHIELD=NUT49_LP_right -DSNIPPET=studio-rpc-usb-uart
west build -d build/left -b nice_nano_v2 -- -DSHIELD=NUT49_LP_left
```

## Flashing

1. Double-tap reset on nice!nano v2 to enter bootloader (NICENANO drive appears)
2. Drag and drop the `.uf2` file onto the drive
3. Flash right side (CENTRAL) first, then left side

## ZMK Studio

Edit your keymap in the browser without reflashing.

1. Connect right side (CENTRAL) via USB
2. Open [zmk.studio](https://zmk.studio/) in Chrome or Edge
3. Select the keyboard and start editing

> ZMK Studio works over USB only. Encoder bindings cannot be edited via ZMK Studio.

## Dependencies

- ZMK: [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk) @ `v0.3-branch`
- PMW3610 driver: [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)

## License

MIT
