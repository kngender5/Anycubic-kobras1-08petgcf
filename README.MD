# Anycubic Kobra S1 OrcaSlicer Profiles + Klipper Config

## OrcaSlicer profiles

Included profile sets:

- Nozzle sizes: `0.2`, `0.4`, `0.6`, `0.8`
- Filaments: `PLA`, `TPU`, `PETG`, `PLA+`, `CR-PLA`, `PETG-CF`
- Firmware flavours: **Marlin** (legacy) and **Klipper**

Profile locations:

| Directory | Contents |
|---|---|
| `orcaslicer/printer/` | Printer profiles — Marlin (`*_Marlin_*`) and Klipper (`*_Klipper_*`) |
| `orcaslicer/filament/` | Filament presets (shared across firmware flavours) |
| `orcaslicer/process/` | Process / quality presets (shared across firmware flavours) |

### Using the Klipper printer profiles

1. Import the matching `Anycubic_Kobra_S1_Klipper_250x250x250_0pX.orca_printer` file into
   OrcaSlicer (**File → Import → Import Configs**).
2. The start/end G-code in each Klipper profile calls the macros defined in
   `klipper/macros.cfg`:
   - **Start:** `PRINT_START BED_TEMP=<temp> EXTRUDER_TEMP=<temp>`
   - **End:** `PRINT_END`

---

## Klipper firmware configuration

Files in `klipper/` are ready-to-use Klipper configurations:

| File | Purpose |
|---|---|
| `klipper/printer.cfg` | Main hardware config — kinematics, steppers, heaters, probe, fans |
| `klipper/macros.cfg` | `PRINT_START`, `PRINT_END`, `PAUSE`, `RESUME`, `CANCEL_PRINT`, filament helpers |

### First-time setup

1. Copy `klipper/printer.cfg` and `klipper/macros.cfg` to your Klipper host
   (`~/printer_data/config/` for Mainsail/Fluidd).
2. Edit `printer.cfg` and replace the `serial:` path with your board's USB ID
   (`ls /dev/serial/by-id/`).
3. Run `PID_CALIBRATE HEATER=extruder TARGET=260` and
   `PID_CALIBRATE HEATER=heater_bed TARGET=80`, then `SAVE_CONFIG`.
4. Run `PROBE_CALIBRATE` to set your Z offset.
5. Tune `pressure_advance` in `[extruder]` for your filament (see
   [Pressure Advance docs](https://www.klipper3d.org/Pressure_Advance.html)).
6. Optionally run `SHAPER_CALIBRATE` with an ADXL345 and update `[input_shaper]`.
