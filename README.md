# idf-sdk-openclu

Prebuilt Arduino-ESP32 SDK libraries (`framework-arduinoespressif32-libs`) for the Panel firmware, with a larger
lwIP socket limit. The package is published as a release asset so PlatformIO can download it through
`platform_packages`; this repository holds no source code.

## Releases

| Release | Package version | Base | Changes vs base |
|---|---|---|---|
| `sdk-esp32-3.3.8-h-panel.6` | `5.5.4+panel.6` | `esp32-core-3.3.8-libs` | as `panel.5` (same ESP-IDF patches), with `CONFIG_HEAP_TASK_TRACKING` off again on esp32s3 (base value) |
| `sdk-esp32-3.3.8-h-panel.5` | `5.5.4+panel.5` | `esp32-core-3.3.8-libs` | as `panel.4`, plus four ESP-IDF `release/v5.5` heap task tracking commits (`a3c4103f` test only, `ce82719e`, `b475fda9`, `0ffb79d3`); `sdkconfig` unchanged |
| `sdk-esp32-3.3.8-h-panel.4` | `5.5.4+panel.4` | `esp32-core-3.3.8-libs` | as `panel.3`, plus on esp32s3 `CONFIG_PM_PROFILING`, `CONFIG_MBEDTLS_EXTERNAL_MEM_ALLOC`, `CONFIG_SPIRAM_ALLOW_BSS_SEG_EXTERNAL_MEMORY`, `CONFIG_HEAP_TASK_TRACKING`, `CONFIG_LWIP_TCP_WND_DEFAULT` 11520, `CONFIG_LWIP_TCP_SND_BUF_DEFAULT` 11488 |
| `sdk-esp32-3.3.8-h-panel.3` | `5.5.4+panel.3` | `esp32-core-3.3.8-libs` | as `panel.2`, plus `CONFIG_LWIP_IP_FORWARD` off (so `CONFIG_LWIP_IPV4_NAPT` and `CONFIG_LWIP_IPV4_NAPT_PORTMAP` are gone) |
| `sdk-esp32-3.3.8-h-panel.1` | `5.5.4+panel.1` | `esp32-core-3.3.8-libs` | high_perf setup reproduced (XIP, -O2, cache I/D 16/32 KB, data line 64 B, `LCD_RGB_RESTART_IN_VSYNC` off), `CONFIG_LWIP_MAX_SOCKETS` 48, `CONFIG_LWIP_MAX_ACTIVE_TCP` 48 |
| `sdk-esp32-3.2.0-h-panel.1` | `5.4.1+panel.1` | `esp32-3.2.0-h` (high_perf) | `CONFIG_LWIP_MAX_SOCKETS` 16 → 48, `CONFIG_LWIP_MAX_ACTIVE_TCP` 16 → 48 |

Chips and memory variants included: `esp32` (`qio_qspi`) and `esp32s3` (`qio_opi`). Other chips and variants are
removed on purpose, so a build for them fails instead of linking mismatched libraries.

## Use with PlatformIO (pioarduino 54.03.20)

```ini
platform = https://github.com/pioarduino/platform-espressif32/releases/download/54.03.20/platform-espressif32.zip
platform_packages =
	platformio/framework-arduinoespressif32-libs@https://github.com/maciejs20/idf-sdk-openclu/releases/download/sdk-esp32-3.2.0-h-panel.1/esp32-3.2.0-h-panel.1.zip
framework = arduino
```

## Provenance

- `sdk-esp32-3.3.8-h-panel.6` (2026-11-16): same base, builder, source pins and ESP-IDF patches as `panel.5`; built by `scripts/build/build_sdk.sh` of `maciejs20/espPlatform` at `600b896` (release 0.21.0). The only `sdkconfig` change vs `panel.5` is esp32s3 `CONFIG_HEAP_TASK_TRACKING` not set (its dependent `CONFIG_HEAP_TRACK_DELETED_TASKS` drops out): with `panel.5` an image still deadlocked in the tracking lock (allocation inside the tracking critical section spinning on the PSRAM heap lock, interrupt watchdog at about 8% of starts). The patches have no effect with tracking off; the IDF version string still reads `v5.5.4-dirty`.
- `sdk-esp32-3.3.8-h-panel.5` (2026-11-15): same base, builder and source pins as `panel.1`; built by `scripts/build/build_sdk.sh` of `maciejs20/espPlatform` at `c3d362a` (release 0.15.1), which applies the patches in its `scripts/build/sdk_patches/esp-idf-v5.5.4/` to ESP-IDF v5.5.4: `a3c4103f` (test only, needed by `0ffb79d3`), `ce82719e` and `b475fda9` (task usage on realloc), `0ffb79d3` (the task tracking lock is a critical section instead of a mutex; with `panel.4` freeing memory inside the task watchdog reconfiguration blocked on that mutex and the image hung at boot). `sdkconfig` identical to `panel.4`; the IDF version string reads `v5.5.4-dirty`.
- `sdk-esp32-3.3.8-h-panel.4` (2026-11-13): same base, builder and source pins as `panel.1`; built by `scripts/build/build_sdk.sh` of `maciejs20/espPlatform` at `8c412d3` (release 0.14.0). The `sdkconfig` changes vs `panel.3` are esp32s3 only: power management profiling, mbedTLS allocations in PSRAM, `EXT_RAM_BSS_ATTR` objects in PSRAM, heap task tracking, TCP window 5760 → 11520 and send buffer 5744 → 11488; esp32 unchanged.
- `sdk-esp32-3.3.8-h-panel.3` (2026-11-11): same base, builder and source pins as `panel.1`; built by `scripts/build/build_sdk.sh` of `maciejs20/espPlatform` at `eae4a41` (branch `op/sdk-panel3`, release 0.11.1). The only `sdkconfig` change vs `panel.2` is `CONFIG_LWIP_IP_FORWARD` not set (its dependents `CONFIG_LWIP_IPV4_NAPT` and `CONFIG_LWIP_IPV4_NAPT_PORTMAP` drop out), so a client of the device's open configuration access point cannot reach its station network.
- `sdk-esp32-3.3.8-h-panel.1`: base `https://github.com/espressif/arduino-esp32/releases/download/3.3.8/esp32-core-3.3.8-libs.tar.xz`; built with `esp32-arduino-lib-builder` `14a0af9`, ESP-IDF `735507283d` (v5.5.4), arduino-esp32 `c6428be92`, tinyusb `eae25b531`. Use with pioarduino 55.03.38; `lib_archive = yes` keeps boards without Wi-Fi free of Wi-Fi.
- `sdk-esp32-3.2.0-h-panel.1`:

- Base package: `https://dl.espressif.com/AE/esp-arduino-libs/esp32-3.2.0-h.zip` (Espressif high_perf build).
- Built with `esp32-arduino-lib-builder` at `a3c4d2c3deba06173c8889c094c212ff359857e9` (`high_perf/v3.2.0`),
  ESP-IDF `2f7dcd862a` (v5.4.1), arduino-esp32 `210edfeaa`, tinyusb `8c1802e41`; managed component versions taken
  from the base `dependencies.lock`.
- Build environment: the builder's own Docker file on `ubuntu:24.04`; overrides appended to
  `configs/defconfig.<chip>`; `./build.sh -t esp32` and `./build.sh -t esp32s3`.
- Comparison with the base (per chip): `sdkconfig` equal except the two overrides; library file list equal;
  `dependencies.lock` equal. `IDF_VER`/`versions.txt` show the same ESP-IDF commit with a one-character longer
  abbreviated hash; `versions.txt` no longer lists four managed components (`eppp_link`, `esp_hosted`,
  `esp_serial_slave_link`, `esp_wifi_remote`) that are not part of the esp32/esp32s3 libraries of the base either.

## Checksums

`esp32-3.3.8-h-panel.6.zip`: sha256 `76d7f4c538a8798fcbb3d51049ec85ac95b7bd56c12853cc0be351e17c05626e`

`esp32-3.3.8-h-panel.5.zip`: sha256 `ca4f4e8654ad20a3e3001a80d847f7c2de7c199458d72df9144bdcae3d582756`

`esp32-3.3.8-h-panel.4.zip`: sha256 `dcf9882ae257082e61d012eb3417f2008fb6673258c68dad105e5d248c7647a0`

`esp32-3.3.8-h-panel.3.zip`: sha256 `b3e6e27ca3eff37659e3a1cb1f008e3201f4313a9522616a19e6f0e759bf8df4`

`esp32-3.2.0-h-panel.1.zip`: sha256 `3e42e872aeed8c5ac9904b0d30b2d42cff40ebc1f5c34eec81e845616d9fc5b3`

## License

The libraries are compiled from ESP-IDF (Apache-2.0), arduino-esp32 (LGPL-2.1) and their components, each under its
own license as shipped in the original Espressif package. No changes to their source code, only configuration.
