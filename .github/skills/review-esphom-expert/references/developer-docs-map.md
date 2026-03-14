# ESPHome Developer Docs Map

Primary documentation source for this skill:

- Site: https://developers.esphome.io/
- Source repository: https://github.com/esphome/developers.esphome.io

Use this documentation first during review.

## What To Check There First

- Architecture overview
- Component architecture
- GPIO, I2C, SPI, UART component architecture pages
- Automations and API architecture
- CI and test expectations
- Contribution and code expectations when reviewing workflow or structural changes

## When To Fall Back To User Docs

Use the user-facing ESPHome docs only if you still need a YAML-level answer such as:

- a concrete configuration option name
- widget property syntax
- supported image format for a widget
- exact sensor or component configuration examples

## Reviewer Guidance

- If a finding is backed by developer docs, say that explicitly.
- If you had to fall back to user docs for a YAML constraint, say that too.
- Do not present undocumented assumptions as defects.