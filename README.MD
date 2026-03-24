# Anycubic Kobra S1 — OrcaSlicer Profiles & Klipper Configuration

## Contents

- [OrcaSlicer profiles](#orcaslicer-profiles)
  - [How to import and change a profile in OrcaSlicer](#how-to-import-and-change-a-profile-in-orcaslicer)
- [Klipper firmware](#klipper-firmware)
  - [How to install and implement Klipper changes](#how-to-install-and-implement-klipper-changes)

---

## OrcaSlicer profiles

Included profile sets:

| Category | Files |
|----------|-------|
| **Printer (Marlin)** | `0.2 mm`, `0.4 mm`, `0.6 mm`, `0.8 mm` brass / hardened steel |
| **Printer (Klipper)** | `0.2 mm`, `0.4 mm`, `0.6 mm`, `0.8 mm` — Klipper start/end G-code |
| **Filament** | `PLA`, `PLA+`, `PETG`, `PETG-CF`, `TPU`, `CR-PLA` |
| **Process** | `0p2_0p08LH`, `0p4_0p20LH`, `0p6_0p30LH` per filament type; `0p8_0p32LH` for PETG-CF |

Profile file locations:

```
orcaslicer/printer/     ← .orca_printer files
orcaslicer/filament/    ← .orca_filament files
orcaslicer/process/     ← .orca_process files
```

---

### How to import and change a profile in OrcaSlicer

#### Step 1 — Download the profile files

Clone or download this repository to your computer:

```bash
git clone https://github.com/kngender5/Anycubic-kobras1-08petgcf.git
```

All profile files live under `orcaslicer/`.

#### Step 2 — Import a printer profile

1. Open **OrcaSlicer**.
2. Click the **Printer** dropdown at the top of the main window and choose
   **"Add/Remove printers"** (or click the ⚙️ gear icon next to the printer
   name).
3. In the Printer dialogue, click **"Import"** (bottom-left).
4. Navigate to `orcaslicer/printer/` and select the `.orca_printer` file that
   matches your nozzle size and firmware:
   - **Marlin firmware** → `Anycubic_Kobra_S1_Marlin_250x250x250_0p4.orca_printer`
   - **Klipper firmware** → `Anycubic_Kobra_S1_Klipper_250x250x250_0p4.orca_printer`
5. Click **Finish** / **Confirm**. The printer now appears in your printer list.

#### Step 3 — Import a filament profile

1. Click the **Filament** dropdown and choose **"Manage filaments"**.
2. Click **"Import"** and navigate to `orcaslicer/filament/`.
3. Select the `.orca_filament` file for your material (e.g.
   `Generic_PLA_Enclosed.orca_filament`).
4. Click **Finish**. The filament profile is now available in the dropdown.

#### Step 4 — Import a process (quality) profile

1. Click the **Process** dropdown and choose **"Manage processes"**.
2. Click **"Import"** and navigate to `orcaslicer/process/`.
3. Select the `.orca_process` file that matches your nozzle and layer height
   (e.g. `PLA_0p4_0p20LH.orca_process`).
4. Click **Finish**.

#### Step 5 — Select active profiles for a print

1. In the main OrcaSlicer window, use the three dropdowns near the top to
   select **Printer**, **Filament**, and **Process**.
2. Load your model (drag-and-drop a `.stl` or `.3mf` file onto the build
   plate).
3. Adjust any per-print settings in the right-hand panel (supports, infill,
   etc.).
4. Click **Slice** then **Export G-code** (or **Print via network** if you have
   OrcaSlicer connected to your printer over the network / Moonraker).

#### Step 6 — Edit and save a profile

1. Select the profile you want to edit from its dropdown.
2. Change any value in the settings panel on the right.
3. Click the **save** icon (💾) next to the profile name to save changes.
   - To keep the original profile intact, click the **copy** icon first to
     create a new profile, then edit and save the copy.
4. To export the modified profile for sharing or backup, click
   **"Export preset"** from the three-dot (⋮) menu next to the profile name.

---

## Klipper firmware

Klipper configuration files for the Anycubic Kobra S1 live under `klipper/`:

```
klipper/printer.cfg    ← hardware config (MCU, steppers, heaters, probes …)
klipper/macros.cfg     ← PRINT_START, PRINT_END, PAUSE, RESUME, helpers …
```

---

### How to install and implement Klipper changes

#### Prerequisites

- Klipper, Moonraker, and a front-end (Mainsail or Fluidd) already installed
  on a Raspberry Pi (or equivalent SBC) connected to the Kobra S1 USB port.
- SSH access to the Pi.

#### Step 1 — Back up your existing configuration

```bash
# On the Raspberry Pi
cp ~/printer_data/config/printer.cfg ~/printer_data/config/printer.cfg.bak
```

Always keep a backup before making changes.

#### Step 2 — Copy the new config files to the Pi

From your computer:

```bash
scp klipper/printer.cfg  pi@<PI_IP>:~/printer_data/config/printer.cfg
scp klipper/macros.cfg   pi@<PI_IP>:~/printer_data/config/macros.cfg
```

Or use the Mainsail / Fluidd **"Upload file"** button in the **Configuration**
tab to upload both files.

#### Step 3 — Edit printer.cfg for your hardware

Open `printer.cfg` and replace every `<placeholder>` with the real pin name
for your board. Common values to update:

| Setting | What to do |
|---------|------------|
| `serial:` under `[mcu]` | Run `ls /dev/serial/by-id/` on the Pi and paste the path |
| `x_offset` / `y_offset` under `[bltouch]` | Measure the probe-to-nozzle offset with a ruler |
| `z_offset` under `[bltouch]` | Perform the paper-test calibration and run `SAVE_CONFIG` |
| `rotation_distance` under `[extruder]` | Calibrate `rotation_distance`; see [Klipper rotation_distance guide](https://www.klipper3d.org/Rotation_Distance.html) |
| `pid_kp/ki/kd` | Run `PID_CALIBRATE HEATER=extruder TARGET=220` then `SAVE_CONFIG` |

#### Step 4 — Check the configuration

In Mainsail / Fluidd, go to **Configuration → printer.cfg** and click
**"Save & restart"**.  Klipper will validate the config and report any errors
in the console.  Fix any reported errors before proceeding.

Alternatively, from SSH:

```bash
systemctl restart klipper
journalctl -u klipper -f          # Watch for errors
```

#### Step 5 — Home and test

1. In Mainsail / Fluidd, open the **Dashboard**.
2. Click **"Home all"** (`G28`).
3. Verify that the printer homes correctly and the probe deploys.
4. Run `BED_MESH_CALIBRATE` to generate the first mesh.
5. After the mesh is complete, run `SAVE_CONFIG` to persist the results.

#### Step 6 — Calibrate the extruder

```
# In the Mainsail / Fluidd console:
PID_TUNE_ALL            # Runs hotend + bed PID autotune, then SAVE_CONFIG
```

After saving, re-home and run a test print to verify everything is working.

#### Step 7 — Configure OrcaSlicer to use Klipper macros

Import one of the Klipper printer profiles from `orcaslicer/printer/`
(see [Step 2 above](#step-2--import-a-printer-profile)).  The profiles already
contain the correct start and end G-code:

```
; Start G-code
PRINT_START BED=[bed_temperature_initial_layer_single] EXTRUDER=[nozzle_temperature_initial_layer]

; End G-code
PRINT_END
```

OrcaSlicer substitutes the actual temperatures at slice time.  No further
start/end G-code changes are needed.

#### Step 8 — Making ongoing Klipper config changes

1. Edit `printer.cfg` or `macros.cfg` in Mainsail / Fluidd
   (**Configuration** tab → click the file name).
2. Make your changes and click **"Save & restart"**.
3. Test the change in the console before printing.
4. Commit the updated file back to this repository so your changes are tracked:

   ```bash
   # On your computer after downloading the updated file
   git add klipper/printer.cfg klipper/macros.cfg
   git commit -m "Update Klipper config: <brief description>"
   git push
   ```

> **Tip:** Keep the `klipper/` folder in this repository in sync with the files
> on your Pi.  This gives you version history and makes it easy to roll back a
> bad change.
