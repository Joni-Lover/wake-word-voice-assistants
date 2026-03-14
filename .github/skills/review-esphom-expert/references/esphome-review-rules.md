# ESPHome Review Rules

Use these rules when reviewing ESPHome changes in this repository.

## General

- Prefer https://developers.esphome.io over memory or inference.
- Use the user-facing ESPHome docs as a fallback only when the developer docs do not cover a YAML-specific option or component limitation.
- Review for build validity, runtime correctness, and hardware conflicts in that order.
- A valid review distinguishes documented requirements from project preferences.

## Packages And Includes

- `packages:` merges included YAML into the current config.
- When list values are extended or prepended, verify the intended merge behavior rather than assuming replacement.
- Factory YAML should include the device config once and only add the extra provisioning/update behavior needed for first-time install.

## I2C And Cross-References

- When multiple I2C buses exist, each consumer should explicitly reference the right `i2c_id` if ambiguity is possible.
- Review new pins for overlap with existing I2C, SPI, audio, display, touchscreen, and strapping-sensitive pins.
- If a bus was renamed, validate every dependent component was updated.

## LVGL And Display

- LVGL requires a configured display with `auto_clear_enabled: false` and no display `lambda`.
- Most LVGL-backed displays should use `update_interval: never` so LVGL owns rendering.
- Review page/widget IDs for consistency with every `lvgl.*.update`, show/hide, or start/stop automation.
- `animimg` is valid for frame-sequence animation; verify all referenced frames exist.
- For `image` widgets in LVGL, `RGB565` images are the safe expected format.

## Labels, Fonts, And Text

- Review `label` `long_mode` usage when the change claims scrolling or wrapping behavior.
- Review fonts for the required glyph coverage if the device must display non-Latin text.
- `bpp` changes affect appearance and memory usage; treat them as intentional and check consistency across related fonts.

## Touchscreen

- For GT911, validate controller wiring and I2C usage.
- If the design uses capacitive keys, a `binary_sensor` with `platform: gt911` and `index` is the documented direct approach.
- If the design uses `on_touch` coordinate hit-testing instead, review the coordinates against the actual layout and treat it as custom logic rather than a built-in key binding.

## Remote Receiver And Transmitter

- Review pin mode, inversion, tolerance, and buffer settings for plausibility.
- Confirm the RX and TX pins do not collide with existing peripherals.

## Sensors

- For ADC-based battery reporting, review the scaling formula, units, and clamp logic.
- For copy sensors, review whether the source sensor update cadence and units remain coherent.
- For environmental sensors, check bus assignment, interval, and variant selection.

## Voice Assistant Automations

- Review event handlers as a state machine.
- Check that every state transition updates the correct widgets and pages.
- Check that labels or globals cleared on exit are also initialized on entry.
- Be careful with race conditions around media playback, wake word restarts, and `wait_until` conditions.