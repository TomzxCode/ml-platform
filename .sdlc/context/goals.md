# Goals

> Horizon: multi-year (a few years). These goals describe the overall vision for the platform, not a single execution period.

## Why This Project Exists

ml-platform exists so MLEs can do machine learning at commercial scale (data processing, training, batch inference, online inference, data transfer) through one unified, low-friction interface, without owning infrastructure concerns.

## Vision

MLEs accomplish all typical ML operations through the platform's CLI alone, with onboarding measured in minutes, while ML infrastructure engineers operate the platform with minimal support load.

## Objectives

### Simplified user experience doing typical ML operations

**Owner:** The implementors of this platform

**Statement:** Any typical ML operation a user needs is doable through the unified interface with minimal friction.

**Key results:**

| Key result | Target | Measurement method | Baseline | Status |
|---|---|---|---|---|
| Users can accomplish any of the typical ML operations by invoking the CLI (not requiring the addition of new commands) | Full coverage of typical ML operations (data processing, training, batch inference, online inference, data transfer) via existing CLI commands | Operation coverage audit: checklist of typical ML operations vs. commands that accomplish them end to end | None (no CLI yet) | Not started |
| Onboarding to any task is quick | Under 1 hour from install to first successful task completion | Time-to-first-success measurement (to be instrumented; e.g., first-run telemetry or usability tests) | None (to be instrumented) | Not started |

## KPIs (ongoing)

| Indicator | Target | Measurement period | Measured where |
|---|---|---|---|
| CLI operation coverage (typical ML operations completable without new commands) | Trend toward 100% | Quarterly | Operation coverage audit |
| Onboarding time to first successful task | Under 1 hour | Continuous | To be instrumented |

## Guiding Principles

- Least amount of friction for MLEs getting their work done.
- Minimal on-call and support burden for ML infrastructure engineers.

## Non-goals

- None stated.

## Feature Alignment

| Feature | Advances objective | Contribution |
|---|---|---|
| (none yet) | | |

## Review Cadence

- **Review frequency:** to be set once execution planning begins
- **Last reviewed:** 2026-08-23
- **Next review:** to be set

## Open Questions

- Onboarding time has no instrumentation yet; define the measurement method when telemetry exists (see `/create-telemetry` on the first feature).
