# Project Review Map

This repository contains ESPHome voice assistant configurations with custom display assets and release workflows.

## Key Files

- `esp32-s3-box-3/esp32-s3-box-3.yaml`: upstream-style baseline for the ESP32-S3-Box-3 configuration
- `esp32-s3-box-3/joni.yaml`: custom Joni variant with display, LVGL, sensors, and integrations
- `esp32-s3-box-3/*.factory.yaml`: factory-install packaging and update/import setup
- `.github/workflows/build.yml`: firmware build matrix and release flow
- `casita/*`: UI images used by display or LVGL pages

## Current Project Invariants

- The repository ships YAML-first device configurations; most important regressions are wiring, IDs, state transitions, or build packaging issues.
- `joni.yaml` is the main custom review target when the task mentions Joni, LVGL, touchscreen, sensor dock, or voice assistant UI changes.
- Factory variants should remain thin wrappers around the main device YAML.
- GitHub workflow changes matter when a new factory image is expected to build and publish.

## High-Value Review Checks For This Repo

- New images added under `casita/` should correspond to actual substitutions or `image:` IDs.
- If animation is introduced, check that both frame assets and `animimg` references exist.
- If a second bus or new peripheral is added, verify the main board peripherals still reference the original bus explicitly.
- If a new factory YAML is added, check it is included in `.github/workflows/build.yml` when the firmware should be shipped.
- If the config changes repository URLs, verify they match the actual GitHub owner, repo, and Pages path used by the project.

## Review Boundaries

- Prefer findings about correctness, regressions, unsupported config, or missing build wiring.
- Do not raise findings for stylistic YAML differences unless they create ambiguity or maintenance risk.
- Hardware behavior such as touch hit areas, image alignment, and sensor calibration should be called out as test gaps unless confirmed by compile or device testing.