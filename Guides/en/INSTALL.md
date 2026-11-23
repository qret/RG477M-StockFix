# Installation & Technical Guide (EN)

## 1. Prerequisites
- **Device:** Anbernic RG477M (MT6897 / Dimensity 8300) running **Stock Firmware V1.32**.
- **Root:** Magisk installed (if your device is still unrooted, follow **[RG477M-Toolkit](https://github.com/qret/RG477M-Toolkit)** for zero-wipe root and full stock backup).

## 2. Installing & Updating

### Method A: Directly on RG477M (Recommended)
1. Download **`RG477M-StockFix.zip`** from **[Releases](https://github.com/qret/RG477M-StockFix/releases/latest)**.
2. Open the **Magisk App → Modules → Install from storage**.
3. Pick `RG477M-StockFix.zip` and reboot.
4. For future releases, Magisk reads `updateJson` automatically and displays an **Update** button inside the **Modules** tab.

### Method B: Over ADB from a PC
```sh
adb push RG477M-StockFix.zip /data/local/tmp/
adb shell su -c "magisk --install-module /data/local/tmp/RG477M-StockFix.zip"
adb shell rm -f /data/local/tmp/RG477M-StockFix.zip
adb reboot
```
*(If `/data/adb/magisk/util_functions.sh` is missing on a fresh headless Magisk install, open the Magisk App once to complete initial environment setup, or use `RG477M-Toolkit` Step 3.)*

## 3. How Quick Panel Modes Behave

### Performance Profiles (`persist.sys.cpu.mode`)
| Quick Panel Mode | CPU Policy | GPU / GED Policy | Screen Off / Idle |
| :--- | :--- | :--- | :--- |
| **`bal` (`1`)** | `schedutil` (`1000 us`), `cpuidle=teo`, `top-app uclamp.min=45` | `1400 MHz` unlocked, GED `margin=130`, `timer=50`, `step=15`, `floor=265 MHz` | Drops to `265 MHz` GPU / `400 MHz` CPU |
| **`game` (`2`)** | `schedutil` (`1000 us`), `uclamp.min=55`, mid-cluster floor `1400 MHz` | `1400 MHz` unlocked, GED `margin=130`, `timer=40`, `step=18`, 3D floor `780 MHz` (`OPP 38`) | Drops to `265 MHz` GPU / `400 MHz` CPU |
| **`turbo` (`3`)** | Stock clocks + `cpuidle=teo`, `uclamp.min=60` | Stock clocks + `margin=130` | Resets cleanly |
| **`saver` (`0`)** | Conservative `schedutil` (`4000 us`), `uclamp=0` | Conservative `margin=530`, `floor=265 MHz` | Minimum idle power |

### Smooth Fan Curves (`fan_mode` in system settings)
- **`bal` (`fan_mode=1`):** `<48°C → 0` · `48–56°C → 65` · `56–64°C → 85` · `64–72°C → 110` · `72–80°C → 145` · `80–85°C → 200` · `>=85°C → 255`.
- **`turbo` (`fan_mode=2`):** `<42°C → 80` · `42–52°C → 115` · `52–60°C → 150` · `60–68°C → 185` · `68–76°C → 220` · `>=76°C → 255`.
- **`Screen OFF`:** Forces `0 PWM` immediately regardless of SoC temperature, yields `chmod 666` on `pwm_fan_set`, and pauses daemon polling so the system enters true deep sleep.
- **`silent` / `fixed`:** Controlled directly by the stock Anbernic quick panel (`chmod 666`).

### Speaker Anti-Pop (`GPIO 5` / `is_fm_enable`)
- Screen ON & no 3.5 mm jack → `1.0` (analog speaker amp stays clean and ready; 3s DAC standby stays intact).
- 3.5 mm headset plugged → `0.1` immediately.
- Screen OFF → `0.1` once audio `runtime_status` leaves `active`.

## 4. Verification & Uninstall
- Check daemon status over ADB:
  ```sh
  adb shell su -c "ps -ef | grep rg477m_perf_daemon"
  ```
- **Uninstall:** Remove the module in **Magisk App → Modules** and reboot, or run:
  ```sh
  adb shell su -c "touch /data/adb/modules/rg477m_perf_tuning/remove"
  adb reboot
  ```
- **Bootloop safety:** `post-fs-data.sh` counts incomplete boots and automatically writes `disable` after 2 consecutive failed boots before `sys.boot_completed=1`.
