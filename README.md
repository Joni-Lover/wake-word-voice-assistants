# ESPHome firmwares

This repo holds the source of various firmwares used for installing ESPHome onto devices with [esphome/esp-web-tools](https://github.com/esphome/esp-web-tools).

## Local Build, Flash, and Debug

1. Install ESPHome matching CI:
   `pip install esphome==2026.2.4`

2. Validate the main config locally:
   `esphome compile esp32-s3-box-3/joni.yaml`

3. Flash the main config over USB on Windows:
   `esphome run esp32-s3-box-3/joni.yaml --device COM3`

4. Monitor serial logs during local debugging:
   `esphome logs esp32-s3-box-3/joni.yaml --device COM3`

5. To test the first-install onboarding path, flash the factory config instead:
   `esphome run esp32-s3-box-3/joni.factory.yaml --device COM3`

6. After flashing, provision Wi-Fi with the device AP, Improv Serial, or BLE onboarding flow.
