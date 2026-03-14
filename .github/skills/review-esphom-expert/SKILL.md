---
name: review-esphom-expert
description: 'Review ESPHome and CI changes in this repository. Use for code review, PR review, regression review, or plan-vs-implementation review involving ESPHome YAML, esp32-s3-box-3 variants, LVGL UI, voice assistant flows, GPIO and I2C assignments, factory packages, images, and GitHub workflow changes. Findings-first, checking https://developers.esphome.io first and then official ESPHome component docs only when needed for YAML-specific constraints.'
argument-hint: 'What should be reviewed: PR, diff, changed files, or plan?'
user-invocable: true
---

# ESPHome Project Review

## What This Skill Does

This skill reviews changes in this repository with an ESPHome-first mindset.

It is specialized for:
- ESPHome YAML correctness and regressions
- LVGL display migrations and widget logic
- Voice assistant event flows and page-state transitions
- GPIO, I2C, SPI, audio, IR, touchscreen, and sensor integration changes
- Factory package wiring and GitHub build workflow updates

Use https://developers.esphome.io as the primary source of truth for review guidance, architecture, component model expectations, GPIO/I2C/SPI behavior, automations, and CI/contribution conventions. Use the user-facing ESPHome component docs only as a secondary source when a YAML-specific option or component constraint is not covered in the developer docs.

## When to Use

Use this skill when the task includes any of these:
- Review a PR or local diff in this repository
- Check whether implementation matches a plan
- Audit `joni.yaml`, factory YAMLs, or display/UI changes
- Validate new ESPHome components or wiring changes
- Review CI/build changes that affect firmware publishing

## Required Review Style

Start with findings, ordered by severity.

For each finding include:
- severity: high, medium, or low
- why it is a problem
- the exact file reference
- the expected fix or the condition that should be verified

After findings, include:
- open questions or assumptions
- a short change summary only if useful
- explicit testing gaps if validation was not performed

If there are no findings, say so directly and still call out residual risk or unverified areas.

## Procedure

1. Identify the review scope.
   - Prefer the current diff against `HEAD` or the target branch.
   - If the user asks for plan-vs-implementation review, load the plan and compare it against the changed files.

2. Classify changed files.
   - ESPHome YAML: `*.yaml`, `*.factory.yaml`
   - Build/release workflow: `.github/workflows/*.yml`
   - UI assets: `casita/*`, `error_box_illustrations/*`, `sounds/*`
   - Supporting documentation only if it affects behavior

3. Read the minimum set of files needed to understand behavior.
   - For `esp32-s3-box-3/joni.yaml`, also compare against `esp32-s3-box-3/esp32-s3-box-3.yaml` when the change replaces or extends existing behavior.
   - For factory changes, always inspect the corresponding `*.factory.yaml` and the build workflow.

4. Validate against ESPHome documentation in this order.
   - First consult `https://developers.esphome.io/` or the `esphome/developers.esphome.io` repository for the touched areas.
   - Focus first-pass checks on architecture, component model, GPIO, I2C, SPI, automations, API, and CI expectations.
   - If the developer docs do not provide the required YAML-level constraint, then consult the official ESPHome component docs for the touched component.
   - Keep doc checks scoped to the actual change: LVGL, display, touchscreen, I2C, image, font, remote receiver/transmitter, ADC, voice assistant, packages.

5. Apply the ESPHome review checklist.
   - Use [references/esphome-review-rules.md](./references/esphome-review-rules.md).

6. Apply repository-specific checks.
   - Use [references/project-review-map.md](./references/project-review-map.md).

7. Distinguish defects from preferences.
   - A defect changes behavior, violates docs, introduces configuration inconsistency, or creates likely runtime/build failures.
   - A preference is styling, naming, or an optional refactor. Do not present preferences as findings.

8. Call out missing validation.
   - If compile, hardware verification, or runtime checks were not run, say so.
   - Do not claim hardware behavior is verified unless it was actually tested.

## Decision Points

### If the diff changes only assets
- Review image naming, expected frame pairs, and references from YAML.
- Check whether added or renamed assets are actually wired into the config.
- Do not invent rendering defects without matching YAML usage.

### If the diff changes ESPHome YAML
- Prioritize build-break and runtime-risk issues over cosmetic observations.
- Validate IDs, cross-references, automations, and pin reuse.
- Check whether substitutions are used or left stale.

### If the diff changes LVGL pages
- Verify the display still satisfies LVGL prerequisites.
- Review widget IDs, page transitions, event handlers, and show/hide logic.
- Check for overlapping UI elements only when the coordinates clearly imply it.

### If the diff changes factory YAML or workflow files
- Confirm includes/imports still resolve.
- Confirm the new target is part of the build matrix if it should ship.
- Confirm release/update URLs remain consistent with the repository owner and Pages path.

## Completion Criteria

The review is complete when all of the following are true:
- Every behavior-affecting changed file has been inspected.
- Touched ESPHome components were checked against ESPHome developer docs first, with user docs used only when needed for YAML-specific details, plus known project invariants.
- Findings are concrete, evidence-based, and tied to file locations.
- Unverified assumptions and test gaps are stated explicitly.

## Output Template

Use this structure:

```markdown
Findings

1. [severity] Brief title — why this is a real issue.
   File reference.
   Expected correction or verification.

Open Questions

1. Question or assumption that affects confidence.

Summary

Short summary only if it adds value.
```

## References

- [ESPHome developer docs map](./references/developer-docs-map.md)
- [ESPHome review rules](./references/esphome-review-rules.md)
- [Project review map](./references/project-review-map.md)