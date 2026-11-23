# Changelog (`RG477M-StockFix`)

## v0.1.3
- Separated module into standalone repository `RG477M-StockFix` with native Magisk `updateJson` auto-update support.
- Preserved module ID `rg477m_perf_tuning` for seamless updates over earlier Toolkit installations.
- Synchronized module versioning (`v0.1.3` / `versionCode=103`).

## v0.1.2
- **Fan & Deep Sleep fix (`rg477m_perf_daemon.sh`):**
  - Fan target PWM is now unconditionally forced to `0 PWM` whenever the screen is off (`scr_on <= 0`), fixing fan spin in sleep when SoC was `>=60 °C` after gaming.
  - Stopped periodic `thermal_zone*` loops and `settings get system fan_mode` IPC calls while screen is off so the kernel enters true `Suspend-to-RAM` deep sleep.
  - Restored `chmod 666` on `/sys/devices/platform/pwm_fan/pwm_fan_set` upon screen-off to avoid conflicts with stock power services.

## v0.1.1
- **Timer-free analog speaker click/pop fix:** replaced `AudioFlinger` standby tweaks with stock kernel `is_fm_enable` control (`GPIO 5`), 3.5 mm headset detection (`pmic-codec-accdet/state`), AFE `runtime_status` sleep handling, and clean `trap` exit restoration.

## v0.1.0
- Initial release (`rg477m_perf_tuning`):
  - Unlocked Mali-G615 `1400 MHz` ceiling in `bal` mode + tuned GED scaling margins.
  - Replaced stock CPU `performance` lock in `game` mode with `schedutil` + `uclamp` + `1400 MHz` mid-cluster floor.
  - Smooth ~10s EMA fan curves for `bal` / `turbo` (`silent` and `fixed` left to stock control).
  - Fixed `SurfaceFlinger` `BLASTBufferQueue` ~1s flickers via `system.prop`.
  - Kept stock `thermal_core` enabled with `>=85 °C` (`<80 °C` hysteresis) thermal guard and 2-bootloop auto-disable.
