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
| FEAT-p1 (Unified ML CLI) | Simplified user experience doing typical ML operations | Delivers the unified command surface through which every typical ML operation is accomplished (KR-1) |
| FEAT-p2 (API Server) | Simplified user experience doing typical ML operations | Executes every CLI operation end to end (contract, persistence, dispatch), making the unified surface functional (KR-1) |

FEAT-p3 to FEAT-p14 are children of FEAT-p1, one per CLI command group, and advance the objective through their parent.

## Review Cadence

- **Review frequency:** to be set once execution planning begins
- **Last reviewed:** 2026-08-23
- **Next review:** to be set

## Open Questions

- Onboarding-time instrumentation: the measurement method is now defined by the per-feature telemetry plans (FEAT-p3, p8, p9, p10) as a funnel from `cli_first_run` through `auth_login_succeeded` to the first successful operation; it lands when those features are implemented.
