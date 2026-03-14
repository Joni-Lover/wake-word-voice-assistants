# GitHub Copilot Instructions — Wake Word Voice Assistants

## Project Overview

This repository contains **ESPHome firmware** for wake-word-enabled voice assistant devices. The firmware integrates with **Home Assistant** and supports both on-device and cloud-based wake word detection. Firmware images are compiled and published as GitHub Releases; a GitHub Pages web installer (powered by `esp-web-tools`) lets users flash devices over USB from a browser.

**Repository:** `Joni-Lover/wake-word-voice-assistants`
**License:** Apache 2.0
**Primary language:** YAML (ESPHome configuration)

---

## Supported Devices

| Directory | Device | Notes |
|-----------|--------|-------|
| `esp32-s3-box-3/` | Espressif ESP32-S3-Box-3 | Primary device; has a 320×240 LCD, ES7210 ADC, ES8311 DAC |
| `esp32-s3-box/` | Espressif ESP32-S3-Box (gen 1) | Older hardware variant |
| `esp32-s3-box-lite/` | Espressif ESP32-S3-Box-Lite | Lite variant (has a display; different form factor from S3-Box-3) |
| `m5stack-atom-echo/` | M5Stack Atom Echo | Small device with SK6812 RGB LED indicator |

Each device folder contains:
- `<device>.yaml` — core firmware configuration (imported as a package).
- `<device>.factory.yaml` — factory/release firmware; wraps the core config and adds OTA via HTTP, dashboard import URL, Improv Serial provisioning, and the update platform. ESP32-S3-Box variants additionally include BLE provisioning (`esp32_improv`) and WiFi/BLE enable-disable hooks; the M5Stack Atom Echo uses only `improv_serial`.

---

## Repository Structure

```
.
├── .devcontainer/           # VS Code dev container (Python 3.10 + ESPHome)
├── .github/
│   ├── actions/build/       # Composite action: esphome compile + manifest
│   ├── ISSUE_TEMPLATE/      # Bug report template
│   ├── workflows/
│   │   ├── build.yml            # Main CI: build firmware on PR / release
│   │   ├── build-firmware.yml   # Reusable workflow: matrix build + combine manifests
│   │   ├── build-minimal.yml    # Minimal Atom Echo build (all jobs disabled with `if: false`; kept for future use)
│   │   ├── yaml-lint.yml        # Strict YAML linting (yamllint)
│   │   ├── stale.yml            # Auto-close stale issues after 30 + 5 days
│   │   └── lock.yml             # Auto-lock inactive PRs/issues after 1 day
│   └── dependabot.yml       # Weekly GitHub Actions dependency updates
├── casita/                  # 320×240 PNG illustrations for the S3-Box LCD
├── error_box_illustrations/ # Error-state illustrations
├── sounds/                  # WAV audio files (timer_finished.wav for Atom Echo)
├── esp32-s3-box-3/
│   ├── esp32-s3-box-3.yaml          # Core firmware (included as a package)
│   └── esp32-s3-box-3.factory.yaml  # Factory firmware entry point
├── esp32-s3-box/
├── esp32-s3-box-lite/
├── m5stack-atom-echo/
├── .yamllint                # yamllint rules (2-space indent, max 1 blank line)
└── .gitignore
```

---

## Firmware Architecture

### Configuration Hierarchy

```
<device>.factory.yaml   ← release entry point (OTA HTTP, update platform, improv)
  └── packages:
        └── <device>.yaml  ← core firmware (hardware drivers, VA pipeline, display)
```

The `factory.yaml` uses `!include` and ESPHome `packages:` to compose the full configuration. It always sets `esphome.project.version: dev`; the CI workflow replaces `version: dev` with the real release tag before building.

### Voice Assistant Pipeline

All supported devices implement the same conceptual pipeline:

1. **Wake word detection** — either on-device (`micro_wake_word` with models `okay_nabu`, `hey_mycroft`, `hey_jarvis`) or streamed to Home Assistant.
2. **Speech-to-text** — handled by Home Assistant (cloud or on-prem Whisper).
3. **Intent processing** — Home Assistant conversation pipeline.
4. **Text-to-speech + announcement** — audio played back through the device speaker via `media_player` (speaker platform).

The user can switch between "On device" and "In Home Assistant" wake word engine location via a `select` entity exposed to Home Assistant.

### Voice Assistant Phase IDs (ESP32-S3-Box-3)

| Phase ID | State |
|----------|-------|
| 1 | Idle |
| 2 | Listening |
| 3 | Thinking |
| 4 | Replying |
| 10 | Not ready (no HA connection) |
| 11 | Error |
| 12 | Muted |
| 20 | Timer finished |

### Display (ESP32-S3-Box-3)

The 320×240 LCD shows state-specific PNG illustrations stored in the `casita/` directory. The `draw_display` script selects the correct page based on the current `voice_assistant_phase` global and WiFi/API connection status.

Illustration URLs are substitution variables pointing at raw GitHub content:
```yaml
idle_illustration_file: https://github.com/Joni-Lover/wake-word-voice-assistants/raw/main/casita/idle_320_240.png
```

### Audio Hardware (ESP32-S3-Box-3)

| Component | ESPHome platform | Pin(s) |
|-----------|-----------------|--------|
| Microphone ADC | `es7210` | I2S: LRCLK=GPIO45, BCLK=GPIO17, MCLK=GPIO2, DIN=GPIO16 |
| Speaker DAC | `es8311` | I2S: DOUT=GPIO15 |
| Backlight | `ledc` | GPIO47 |
| Button | `gpio` | GPIO0 (pull-up, inverted) |

### Audio Hardware (M5Stack Atom Echo)

| Component | ESPHome platform | Pin(s) |
|-----------|-----------------|--------|
| Microphone | `i2s_audio` PDM | I2S: LRCLK=GPIO33, BCLK=GPIO19, DIN=GPIO23 |
| Speaker | `i2s_audio` | DOUT=GPIO22 |
| LED | `esp32_rmt_led_strip` SK6812 | GPIO27 |
| Button | `gpio` | GPIO39 (inverted) |

---

## Key ESPHome Components Used

- **`micro_wake_word`** — on-device wake word detection (TFLite models).
- **`voice_assistant`** — manages the full VA pipeline lifecycle (listening → STT → intent → TTS).
- **`media_player` (speaker platform)** — plays TTS announcements and timer sounds.
- **`display` / `display.pages`** — page-based LCD rendering.
- **`script`** — reusable action sequences (`draw_display`, `start_wake_word`, `stop_wake_word`).
- **`select`** — exposes wake word engine location choice to Home Assistant.
- **`switch`** — `mute`, `use_listen_light`, `timer_ringing` controls.
- **`improv_serial` / `esp32_improv`** — Wi-Fi provisioning in factory firmware.
- **`ota` (http_request platform)** — over-the-air firmware updates from GitHub Releases.
- **`update` (http_request platform)** — exposes firmware update entity in Home Assistant.
- **`dashboard_import`** — allows the device config to be imported into the ESPHome Builder.

---

## CI / CD Workflows

### `build.yml` — Main Build Workflow

**Triggers:** Pull requests, manual dispatch (`workflow_dispatch`), published releases.

**Jobs:**
1. **`build-firmware`** — calls `build-firmware.yml` to compile `esp32-s3-box-3/esp32-s3-box-3.factory.yaml` and `m5stack-atom-echo/m5stack-atom-echo.factory.yaml`.
2. **`upload-to-release`** — uploads compiled artifacts to the GitHub Release (release events only).
3. **`deploy-pages`** — generates `index.html` + per-device `manifest.json` files and deploys to GitHub Pages (non-prerelease events only).

### `build-firmware.yml` — Reusable Build Workflow

**Inputs:** `files` (newline-separated YAML paths), `esphome-version`, `release-summary`, `release-url`, `release-version`, `combined-name`.

**Jobs:**
1. **`prepare`** — converts `files` input to a JSON array, generates a version string (`dev-YYYYMMDD-HHMM` or release tag), creates a random artifact-name prefix.
2. **`build`** — matrix job: checks out code, replaces `version: dev` with the real version, invokes `.github/actions/build`, moves output under `output/<version>/`, uploads artifact.
3. **`combine`** — (only when `combined-name` is set) downloads prefixed artifacts, merges `manifest.json` files with `jq`, uploads a single combined artifact.

### `yaml-lint.yml`

Runs `yamllint --strict .` on push to `main` and on PRs whenever `*.yaml` or `*.yml` files change. Configuration is in `.yamllint`.

### `stale.yml`

Marks issues stale after **30 days** of inactivity; closes them after **5 more days**. Issues labelled `not-stale` are exempt.

### `lock.yml`

Locks PRs after **1 day** of inactivity and issues after **1 day**. PRs/issues labelled `keep-open` are exempt. Runs daily at 19:00 UTC.

---

## Development Guidelines

### Adding or Modifying Firmware

1. **Edit the core `<device>.yaml`** for hardware, driver, and pipeline changes.
2. **Edit `<device>.factory.yaml`** only for release-specific settings (update manifest URL, dashboard import URL, project metadata).
3. Keep substitution variables at the top of the YAML for all configurable values (illustration URLs, colour codes, phase IDs).
4. Use `packages:` for code reuse rather than duplicating YAML between devices.
5. The `min_version` field must be updated when using new ESPHome features; currently set to `2025.5.0`.

### YAML Style Rules (`.yamllint`)

- **2-space indentation** — no tabs.
- **Maximum 1 blank line** between blocks; no blank lines at the start of a file (up to 1 allowed at end).
- **`---` document start** is optional (disabled in rules).
- **Line length** is not enforced.
- **Truthy** values (`yes`/`no`) are allowed.
- Validate locally: `yamllint --strict .`

### Adding a New Device

1. Create a new directory `<device-name>/`.
2. Create `<device-name>.yaml` with hardware configuration and the VA pipeline.
3. Create `<device-name>.factory.yaml` following the pattern in `esp32-s3-box-3.factory.yaml` — include `packages:`, `esphome.project`, `ota`, `update`, `http_request`, `dashboard_import`, `wifi`, `improv_serial`, `esp32_improv`.
4. Add the factory YAML path to the `files:` input in `build.yml`.
5. Update the device dropdown in `.github/ISSUE_TEMPLATE/bug_report.yml`.

### Illustrations and Assets

- LCD illustrations: 320×240 PNG, placed in `casita/`, referenced via raw GitHub URLs.
- Timer audio: WAV files in `sounds/` (Atom Echo), FLAC files referenced from the `esphome/home-assistant-voice-pe` repository (S3-Box-3).
- When adding new images, update the substitution variables in the corresponding `<device>.yaml`.

### Update Manifest URL

The `update` component's `source:` URL points to GitHub Pages:
```
https://joni-lover.github.io/wake-word-voice-assistants/<device>/manifest.json
```
This URL must match the directory structure generated by the `deploy-pages` job in `build.yml`. The `ota` block uses the `http_request` platform but does not define its own source URL.

---

## Security Considerations

- The build workflows (`build-firmware.yml`, `build.yml`) pin `actions/checkout`, `actions/upload-artifact`, and `actions/download-artifact` to full commit SHAs to prevent supply-chain attacks. Automation-only workflows (`yaml-lint.yml`, `stale.yml`, `lock.yml`) use version tags instead.
- `dependabot.yml` keeps GitHub Actions dependencies up to date weekly.
- Secrets (Wi-Fi credentials, API keys) are **never committed**; `secrets.yaml` is in `.gitignore`.
- The `build` job has `permissions: contents: read` (minimal scope).

---

## Local Development

The `.devcontainer/devcontainer.json` provides a ready-to-use environment:
- **Base image:** `mcr.microsoft.com/vscode/devcontainers/python:0-3.10`
- **Post-create:** `pip3 install esphome`
- **VS Code extension:** `ESPHome.esphome-vscode` with local validator
- **Privileged USB access** for flashing via the container

To validate a config locally:
```bash
esphome config esp32-s3-box-3/esp32-s3-box-3.yaml
```

To compile locally:
```bash
esphome compile esp32-s3-box-3/esp32-s3-box-3.factory.yaml
```

---

## Useful References

- [ESPHome documentation](https://esphome.io/)
- [ESPHome Voice Assistant component](https://esphome.io/components/voice_assistant.html)
- [ESPHome micro_wake_word component](https://esphome.io/components/micro_wake_word.html)
- [esp-web-tools](https://github.com/esphome/esp-web-tools) — browser-based firmware installer
- [Home Assistant Voice Assistant docs](https://www.home-assistant.io/voice_control/)
- [ESPHome issue tracker](https://github.com/esphome/issues)
