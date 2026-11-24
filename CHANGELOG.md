# Changelog (`RG477M-StockFix`)

## v0.1.5
- **Strict Manual Fan Lock (`fan_mode=3`/`4`):**
  - When manual/custom fan mode is selected, the daemon now takes `chmod 644` ownership of `pwm_fan_set` and holds the exact user-chosen speed (`custom_fan_speed`, `PWM = round(pct * 255 / 100)`) while screen is on (`0 PWM` in sleep), eliminating Anbernic Control's stock `>= 65 °C -> 100% (PWM 255)` jump.
- **Instant Wake-Up & No Post-Sleep Freeze (`rg477m_perf_daemon.sh`):**
  - **Restored wake-up order (`fast_wake_boost`):** on screen-on from `screen_off`, `policy4/7` `700 MHz` caps (`3200/3350 MHz`, or `2600/2800 MHz` when `in_thermal=1`), `UFS` (`simple_ondemand`), `schedutil/rate_limit_us=1000`, and `top-app uclamp` are restored first in ~1–2 ms before `unfreeze_sleep_apps`, `thermal_zone*` polling, and `getprop`.
  - **Reduced sleep poll interval to `1s`:** changed `loop_sleep` in `screen_off` from `3s` to `1s` (ordinary `CLOCK_MONOTONIC` `sleep` does not wake the SoC from `Suspend-to-RAM`, while cutting max wake detection delay 3x).
  - **Dynamic `CATEGORY_HOME` exemption:** resolved active home launcher (`cmd package resolve-activity ... CATEGORY_HOME`) and added its UID to `EXEMPT_UIDS`, preventing third-party frontends/launchers (ES-DE, Daijisho, Beacon, Launcher3, etc.) from freezing during sleep.
  - **Forkless sysfs & wake IPC:** pre-cached matching `thermal_zone*` nodes at startup (`init_thermal_zones`), replaced `$(cat ...)` subshell forks in `get_max_soc_temp()` and `need_reapply` with shell `read -r`, and skipped heavy `settings get system fan_mode` IPC on the wake-up tick.
  - **Fixed `mode_3` (Turbo) recovery:** restored `apply_cpu_schedutil 1000 400000`, `FIX_OPP=-1`, `timer_base_dvfs_margin=40`, and `loading_base_dvfs_step=18` when returning to `mode_3` after sleep/saver.

## v0.1.4
- **Tier 1 & Tier 2 Power Optimization + Config (`/data/adb/rg477m_stockfix.conf`):**
  - Added user-configurable toggle file `/data/adb/rg477m_stockfix.conf` (auto-created on first boot with all options enabled by default, plus `FREEZE_WHITELIST` for custom package/UID exclusions).
  - **Tier 1 (Sched / Memory / UFS / Loggers):**
    - Clamped background cgroups (`background` `uclamp.max=30`, `system-background` `uclamp.max=50`) so background tasks cannot spike performance cores.
    - Enforced `700 MHz` mid/big cluster CPU ceiling and `powersave` UFS governor (`112b0000.ufshci`) in `screen_off` and `saver` (`mode_0`), cleanly restoring full clocks (`3200/3350 MHz`, `simple_ondemand`) on wake/active profiles without fighting stock `thermal_core`.
    - Added CPU ceiling backoff (`2600/2800 MHz` on `policy4`/`policy7`) inside `thermal_guard >= 85 °C` for faster cooldown alongside the `900 MHz` GPU cap.
    - Tuned GKI 6.1 memory & UFS (`lru_gen/min_ttl_ms=1000`, `vm.swappiness=60` for 12 GB RAM, `read_ahead_kb=256`, `iostats=0`).
    - Set `persist.traced.enable=0` in `system.prop` and stopped safe non-HAL loggers (`mobile_log_d`, `emdlogger`, `traced`, `traced_probes`), leaving `connsyslogger` and `lbs_dbg_ext` running.
    - Disabled unnecessary stock power-wake settings (`mobile_data_always_on=0`, `doze_pulse_on_pick_up=0`, `wake_gesture_enabled=0`).
  - **Tier 2 (Deep Sleep / Doze / Optional Non-Cellular Toggles):**
    - Enabled and accelerated Android Doze (`dumpsys deviceidle enable all` + faster `device_idle_constants` transition to idle).
    - Added GammaOS-style `cgroup.freeze` sleep freezer for user/background UIDs (`uid >= 10000`) 15s after screen-off (auto-excluding active VPN `tun*` owners such as AdGuard/VPNs, user Doze whitelist entries, active speaker playback, SystemUI, Anbernic controls/launcher/IME, Magisk, and `FREEZE_WHITELIST`), with immediate unfreeze on screen-on and cleanup.
    - Kept `STOP_GPS_DAEMONS=0` and `STOP_RIL_DAEMON=0` disabled by default in `/data/adb/rg477m_stockfix.conf` (stopping Stock V1.32 GNSS/LBS AIDL HALs causes `system_server` Watchdog bootloops).

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
