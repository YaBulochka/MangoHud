# Fork notes: per-GPU HUD layout + APU sensor fixes

Personal fork of [MangoHud](https://github.com/flightlessmango/MangoHud), based on
**v0.8.4**. Branch: `hud-layout`.

Made for a two-GPU desktop: AMD Ryzen 5 9600X (Radeon 610M / gfx1036 iGPU) and a
Radeon RX 7600 (dGPU). MangoHud rendered both GPUs as one block, and the iGPU showed
useless `0 °C` sensors because it has no VRAM or junction temperature sensor.

## Changes

1. **Per-GPU elements**
   `gpu_stats:N`, `vram:N` and `proc_vram:N` render only the GPU at index `N`
   (the index is the position in MangoHud's selected GPU list). Plain `gpu_stats`,
   `vram` and `proc_vram` still render all GPUs, so the old behaviour is preserved.

   ```
   gpu_stats:0
   vram:0

   gpu_stats:1
   vram:1
   ```

2. **`vram_proc`**
   Adds the process VRAM (PVRAM) as a separate value in the VRAM row.

3. **Two-line VRAM rows**

   ```
   VRAM0   5.8 GiB   5.8 GiB
           1124 MHz   64 °C
   ```

   First line: total VRAM in use and process VRAM.
   Second line: memory clock and memory temperature.

4. **APU memory temperature**
   The iGPU has no VRAM temperature sensor. For APUs the VRAM row now uses the
   system RAM sensor (`ram_temp` / `mem_temp`) when available, and omits the value
   when there is no sensor.

5. **APU junction temperature hidden**
   APUs reported `0 °C Jnc`. Junction temperature is now only shown for non-APU GPUs.

6. **`vram_no_label`**
   Hides the `VRAM0` / `VRAM1` label while keeping the table columns, so the values
   stay in place.

## Example config

```
legacy_layout=false
table_columns=3

time
gpu_text=RX 7600,RX 610M

gpu_stats:0
vram:0

gpu_stats:1
vram:1
vram_proc
vram_no_label

cpu_stats
ram
procmem
fps
frametime
```

## Build

```
meson setup build --prefix=/usr \
  -Dmangoapp=true -Dmangohudctl=true \
  -Dwith_xnvctrl=disabled -Dwith_nvml=disabled -Dtests=disabled
ninja -C build
sudo ninja -C build install
```

## Limitations

- PVRAM stays `0.0` on APUs: the driver does not expose per-process VRAM for the iGPU.
- Only the 64-bit build is patched; `lib32-mangohud` is not.
- Based on v0.8.4. Upstream has moved on to 0.8.5-rc, so this branch is not a drop-in
  replacement for newer upstream releases.

## Upstream

https://github.com/flightlessmango/MangoHud

This is an unofficial personal fork, not affiliated with the MangoHud maintainers.
