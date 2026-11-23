# RG477M StockFix

> **Anbernic RG477M (MT6897 / Dimensity 8300 · Stock V1.32)** · [Русская версия ниже](#rg477m-stockfix-ru)

A standalone Magisk module (**id: `rg477m_perf_tuning`**) for **Stock V1.32** on Anbernic RG477M with native Magisk **`updateJson`** auto-update support. Unlocks smooth 60 FPS gaming in everyday modes, calms down fan curves with true silent sleep, eliminates analog speaker clicks/pops, and fixes 1-second display flickers — without disabling thermal protection or flashing a custom ROM.

> Need root first? Use **[RG477M-Toolkit](https://github.com/qret/RG477M-Toolkit)** for zero-wipe backup, Magisk v31.1 installation, and 1-click rollback.

### What It Fixes & Improves
- **Full 60 FPS in Balance mode:** Removes the stock `900 MHz` Mali-G615 GPU cap (unlocks full `1400 MHz`) and tunes GED scaling margins (~33% lower GPU frame times in heavy 3D tests), while idle clocks still drop to `265 MHz` GPU / `400 MHz` CPU.
- **Less heat in Game mode:** Replaces the stock CPU `performance` lock with fast `schedutil` (`rate_limit_us=1000`) + `uclamp` tuning and a `1400 MHz` mid-cluster floor.
- **Smooth EMA fan curves + true sleep shutoff:** Replaces erratic stock `bal` and `turbo` fan curves ("fan hunting") with ~10s EMA-smoothed temperature tracking and gentle PWM ramps. On screen off, the fan cuts to `0 PWM` immediately and daemon polling pauses so the kernel enters true `Suspend-to-RAM`. Stock `silent` and `fixed` modes remain handed back cleanly (`chmod 666`).
- **Timer-free analog speaker click/pop fix:** Holds `/sys/devices/platform/11050000.mt6897-afe-pcm/is_fm_enable` enabled while the screen is on and no 3.5 mm jack is plugged — eliminating `GPIO 5` pops on every 3s `AudioFlinger` standby without touching audio standby timers or battery drain in sleep.
- **Display frame pacing & safety:** Sets `debug.sf.disable_backpressure=0` and MT6897 SurfaceFlinger properties to eliminate ~1s `BLASTBufferQueue` flickers, keeps `thermal_core` active at all times with a `>=85 °C` GPU/boost backoff, and auto-disables on 2 failed boots.

### Installation & Auto-Updates
1. Download **`RG477M-StockFix.zip`** from **[Releases](https://github.com/qret/RG477M-StockFix/releases/latest)**.
2. Open **Magisk App → Modules → Install from storage**, select `RG477M-StockFix.zip`, and reboot.
3. Future updates appear natively inside **Magisk App → Modules** (via `updateJson`) — no PC required.

### Guides & Documentation
- [Installation, Fan Curves & Troubleshooting Guide (EN)](Guides/en/INSTALL.md)
- [Changelog](CHANGELOG.md)
- [Credits & Licences (THANKS.md)](THANKS.md) · Licensed under [MIT](LICENSE)

---

# RG477M StockFix (RU)

> **Anbernic RG477M (MT6897 / Dimensity 8300 · Сток V1.32)**

Самостоятельный Magisk-модуль (**id: `rg477m_perf_tuning`**) для стоковой прошивки **V1.32** на Anbernic RG477M со встроенной поддержкой автообновлений Magisk (**`updateJson`**). Возвращает стабильные 60 FPS в обычных режимах, убирает «качели» вентилятора и завывания во сне, устраняет щелчки динамиков и периодические мерцания экрана — без отключения штатной термозащиты и без смены прошивки.

> Нужен root и безопасный бэкап? Используйте **[RG477M-Toolkit](https://github.com/qret/RG477M-Toolkit)** — установка Magisk v31.1 без сброса данных и откат на сток в 1 клик.

### Что исправляет и улучшает модуль
- **Ровные 60 FPS в режиме «Баланс»:** снимает стоковый потолок `900 МГц` с GPU Mali-G615 (открывает полные `1400 МГц`) и настраивает GED-скейлинг (~на треть ниже время кадра GPU в тяжёлом 3D), при этом в простое частоты сбрасываются до `265 МГц` GPU / `400 МГц` CPU.
- **Меньше нагрева в режиме `Game`:** заменяет стоковую фиксацию CPU в `performance` на быстрый `schedutil` (`1000 мкс`) + `uclamp` и базовую полку `1400 МГц` для mid-кластера.
- **Плавный кулер + тишина во сне:** сглаживает скачки температуры фильтром EMA (~10 с) в режимах `bal` и `turbo`. При выключении экрана вентилятор мгновенно отключается (`0 PWM`), а тяжёлый опрос датчиков и `settings get` замораживается, позволяя консоли полноценно уходить в глубокий сон (`Suspend-to-RAM`). Режимы `silent` и `fixed` полностью отдаются штатной шторке (`chmod 666`).
- **Аппаратный фикс щелчков динамиков (без таймеров):** удерживает `is_fm_enable` при активном экране и свободном 3.5 мм джеке — убирает щелчки от дёрганья `GPIO 5` каждые 3 секунды простоя `AudioFlinger`, при этом в сне и при подключении наушников усилитель обесточивается автоматически.
- **Плавность кадра и защита:** исправляет `debug.sf.disable_backpressure=0` (убирает ~1-секундные мерцания `BLASTBufferQueue`), оставляет штатный `thermal_core` включённым, приглушает частоты при нагреве `>=85 °C` и автоматически отключается после 2 неудачных загрузок.

### Установка и автообновления
1. Скачайте **`RG477M-StockFix.zip`** со страницы **[Releases](https://github.com/qret/RG477M-StockFix/releases/latest)**.
2. Откройте **Magisk → Модули → Установить из хранилища**, выберите архив и перезагрузите консоль.
3. Новые версии будут приходить прямо в приложении **Magisk → Модули** (по кнопке «Обновить») — компьютер больше не нужен.

### Подробные руководства
- [Подробное руководство по установке, кривым кулера и удалению (RU)](Guides/ru/INSTALL.md)
- [История изменений (CHANGELOG)](CHANGELOG.md)
- [Благодарности и лицензии (THANKS.md)](THANKS.md) · Лицензия: [MIT](LICENSE)
