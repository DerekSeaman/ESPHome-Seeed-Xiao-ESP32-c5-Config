# v1.1 Release Notes

## No more copying files

The biggest change in this release: you can now set up a Seeed XIAO ESP32-C5 device entirely from the **ESPHome Device Builder** UI in Home Assistant. Create the device, pick the Seeed Studio XIAO ESP32C5 board, and paste a small `packages:` block into the generated YAML — that's it. No more needing to dig through the Home Assistant ESPHome add-on directory and uploading files manually.

Changes:

- The base configuration is pulled straight from GitHub at build time — no files to copy.
- The `wifi_helpers.h` helper file has been eliminated entirely.
- Brand new step-by-step instructions (with screenshots) in the updated [README](README.md).

## The configuration files have been reorganized

If you're upgrading from an earlier version, the file layout has changed significantly. Since everything is now pulled from GitHub at build time, there's no need to copy any files manually.	The directory structure has also been simplified.

The four configs now available:

- **`Seeed XIAO ESP32-c5 base.yaml`** — the standard configuration: Wi-Fi, sensors, status LED. No Bluetooth proxy.
- **`Seeed XIAO ESP32-c5 proxy.yaml`** — everything in `Seeed XIAO ESP32-c5 base.yaml`, plus Bluetooth proxy/BLE tracking with selectable scan profiles (Low/Medium/High).
- **`Seeed XIAO ESP32-c5 IRK.yaml`** — a lightweight companion for my separate [IRK Capture package](https://github.com/DerekSeaman/irk-capture).
- **`Seeed XIAO ESP32-c5 remote.yaml`** — the small snippet you paste into ESPHome Device Builder to pull in whichever of the above you want.

## Other improvements

- Switched to the dedicated `seeed_xiao_esp32c5` board definition (previously used a generic devkit fallback).
- Refreshed screenshots throughout the README to reflect the current setup.

## Upgrading from an earlier version

1. Remove `wifi_helpers.h` from your ESPHome `common/` folder — it's no longer needed.
2. Follow the new setup steps in the [README](README.md) to recreate your device via ESPHome Device Builder, using the `Seeed XIAO ESP32-c5 remote.yaml` snippet in place of your old device YAML's `packages:` block.
3. Pick `Seeed XIAO ESP32-c5 base.yaml` or `Seeed XIAO ESP32-c5 proxy.yaml` depending on whether you want Bluetooth proxy functionality.
