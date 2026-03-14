# ESPHome firmwares

This repo holds the source of various firmwares used for installing ESPHome onto devices with [esphome/esp-web-tools](https://github.com/esphome/esp-web-tools).

## Local Build, Flash, and Debug

1. Install ESPHome matching CI:
   `pip install esphome==2026.2.4`

2. Create local secrets file from `secrets.yaml.example`:
   `copy secrets.yaml.example secrets.yaml`
   Then fill in your Wi-Fi credentials.

3. Validate and compile locally:
   `esphome compile esp32-s3-box-3/joni.yaml`

4. Flash locally (USB or OTA when available):
   `esphome run esp32-s3-box-3/joni.yaml`

5. Monitor runtime logs:
   `esphome logs esp32-s3-box-3/joni.yaml`
