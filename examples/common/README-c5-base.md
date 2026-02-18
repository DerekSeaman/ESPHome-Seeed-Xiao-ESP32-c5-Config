# Seeed XIAO ESP32-C5 — C5 Base include (plain-English guide)

Note: this project uses a project-level `README.md` at the repository root. For the high-level overview and ESPHome Builder setup instructions, see `../README.md`.

This file documents the `Seeed xiao ESP32-c5 base.yaml` package used by device YAMLs in this repo. It explains each section and option in plain English for ESPHome builders so you can quickly understand, customize, and test C5 devices.

Summary

- Purpose: provide a reusable base configuration for Seeed XIAO ESP32‑C5 boards (esp32c5 variant) so device YAMLs can be small and device-specific.
- Typical usage: a device YAML uses `packages: device: !include "common/Seeed xiao ESP32-c5 base.yaml"` and provides substitutions like `device_name`, `friendly_name`, `api_key`, and `ota_password`.

Key sections and what they do

- `esp32`

  - `variant: ESP32C5`, `board: esp32-c5-devkitc-1`, and `flash_size: 8MB`: selects the C5 hardware, board definition, and explicitly sets the flash to match the Seeed XIAO ESP32-C5's onboard 8 MB flash chip.

  - `framework: esp-idf` and `sdkconfig_options`: enables dual-band Wi‑Fi 6 support (`CONFIG_SOC_WIFI_HE_SUPPORT`, `CONFIG_ESP_WIFI_11AX_SUPPORT`, `CONFIG_SOC_WIFI_SUPPORT_5G`). Leave these unless you need to change SDK behaviour.

- `esphome`:

  - `name: ${device_name}` and `friendly_name: ${friendly_name}`: the base config now includes these, so device YAMLs no longer need to define the `esphome:` section — just provide the substitutions.

- `logger`:

  - `level: DEBUG`, `baud_rate: 115200`, `hardware_uart: USB_SERIAL_JTAG` — controls serial logging level and port. Change `level` to `INFO` or `WARN` on stable devices to reduce logs.

- `status_led`:

  - Configures the board LED pin (GPIO27, yellow USER LED). LED behavior: solid = all OK, slow blink = Wi-Fi connected but no API client, fast blink = no Wi-Fi.

- `api`:

  - `encryption.key: ${api_key}`: expects a substitution named `api_key` (provided by each device YAML or secrets). This enables encrypted API communications to Home Assistant.

  - `reboot_timeout`: auto-reboot behaviour if API is stuck.

- `ota`:

  - Uses `${ota_password}` for OTA updates. Device YAML should provide this substitution or you can use `!secret` values.

- `wifi`:

  - Domain and `use_address` set mDNS/host naming (e.g., `${device_name}.local`).

  - `power_save_mode: NONE`: disables WiFi power saving for better BLE performance and more consistent connectivity.

  - `fast_connect` and `enable_on_boot` are convenience flags for reconnect behaviour.

  - `ssid`/`password` use `!secret wifi_ssid` and `!secret wifi_password` in the base file — ESPHome Builder manages these automatically. A commented `band: "5GHz"` option is included to force 5 GHz only if needed.

  - `ap` sets a fallback captive AP with its own `ssid` and `password` (`wifi_captive`).

  - `on_disconnect`: increments a counter (`_wifi_disconnects_since_boot`) each time Wi-Fi disconnects, exposed via a template sensor for diagnostics.

- `captive_portal`, `mdns`:

  - Standard ESPHome helpers. `captive_portal` allows fallback setup; `mdns` enables local name discovery.

- Bluetooth / BLE

  - `esp32_ble_tracker` and `bluetooth_proxy`: enable BLE scanning and proxying for Home Assistant presence detection.

  - BLE scan parameters are configurable via a `select` entity with three profiles:
    - **Low**: 320ms interval, 30ms window (9% duty cycle) — minimal power consumption
    - **Medium** (default): 320ms interval, 90ms window (28% duty cycle) — balanced performance
    - **High**: 320ms interval, 160ms window (50% duty cycle) — maximum presence detection accuracy
    - Profile selection persists across reboots via `restore_value: true`

- `sensor` / `text_sensor` / `time` / `globals`:

  - Adds common sensors: uptime (converted to hours), internal temperature, Wi‑Fi RSSI, Wi‑Fi info (BSSID, IP, SSID, MAC), Wi-Fi disconnects (since boot), and SNTP time source.

  - `globals`: defines `_wifi_disconnects_since_boot` counter (not restored on reboot) tracked by the Wi-Fi `on_disconnect` handler.

- Antenna control:

  - The C5 uses a hardware-managed LFD182G45DCHD277 RF switch with dedicated `ANT_2G` and `ANT_5G` pins. No `switch:` or `output:` blocks are needed — there is no software-controlled antenna toggle on the C5.

Substitutions and secrets

- The base file relies on substitutions passed in by device YAMLs: commonly `device_name`, `friendly_name`, `${api_key}`, and `${ota_password}`.

- ESPHome Builder automatically manages Wi-Fi secrets (`wifi_ssid`, `wifi_password`, `wifi_captive`) via its built-in secrets storage. Device examples in this repo may include inline dummy `api_key` and `ota_password` values for documentation — replace them before production use.

How device YAMLs use the base

- Example pattern (device YAML):

  ```yaml
  substitutions:
    device_name: esphomec5-garage
    friendly_name: Garage C5
    api_key: "..."
    ota_password: "..."

  packages:
    device: !include "common/Seeed xiao ESP32-c5 base.yaml"
  ```

  Note: The `esphome:` section is no longer needed in device YAMLs — the base config now includes `name:` and `friendly_name:` using the substitutions you provide.

Customization tips

- To change board-specific settings (pins, outputs, sensors), edit the base file only if the change applies to all devices that include it. Otherwise override or extend in the device YAML.

- Antenna switching is hardware-managed on the C5 — no per-device customization needed.

- Reduce `logger` level from `DEBUG` to `INFO` in production to reduce serial noise.

Using with ESPHome Builder

This package is designed for ESPHome Builder in Home Assistant, which automatically handles:
- Wi-Fi secrets storage
- Firmware compilation
- Device flashing and OTA updates

See the main `README.md` for complete ESPHome Builder setup instructions.

Notes and cautions

- `sdkconfig_options` toggles low-level SDK flags — only change these if you know the SDK consequences.

- The base uses `!secret` for Wi‑Fi values which ESPHome Builder manages automatically.

- The `api.encryption.key` and `ota.password` should be unique per device — ESPHome Builder can generate these for you.
