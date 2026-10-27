# Release Notes — Checkout Service

This document provides structured release notes for each tagged version in the repository. Release notes bridge Git commits and production deployments, turning raw commits into transparent, auditable business and operational context.

---

## Release v1.1.1

- **Release Date**: 2026-05-04
- **Git Tag**: [`v1.1.1`](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill) (Annotated)
- **Target Commit**: `82535d9`
- **Deployment Target**: Production
- **Rollback Target**: **`v1.1.0`** (Commit: `86989b6`)

### Summary of Changes
This patch release stabilizes checkout service runtime configuration, introduces explicit service naming constants, and refactors application entrypoint logic for improved observability.

#### Added
- Added `SERVICE_NAME = 'checkout-service'` constant in `src/app.js` to ensure consistent service identity across log collectors.
- Added structured startup log message outputting service name and `SERVICE_VERSION`.

#### Changed
- Refactored application execution entrypoint from `runCheckout()` to standard `run()`.
- Standardized placeholder version definition to `SERVICE_VERSION = '0.0.0-placeholder'`.

#### Fixed
- Fixed unstandardized service logging format, enabling container orchestrators to verify service startup.

### Operational Guidance & Rollback Plan
- **Pre-deployment Verification**: Execute `node src/app.js` and verify output indicates service startup with valid `SERVICE_NAME`.
- **Rollback Procedure**: If runtime exceptions or regressions occur post-deployment:
  ```bash
  git checkout v1.1.0
  # Redeploy known-good v1.1.0 container/artifact
  ```

---

## Release v1.1.0

- **Release Date**: 2026-04-10
- **Git Tag**: [`v1.1.0`](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill) (Annotated)
- **Target Commit**: `86989b6`
- **Deployment Target**: Staging / Production Documentation Promotion
- **Rollback Target**: **`v1.0.0`** (Commit: `ff1fb9a`)

### Summary of Changes
This minor release introduces operational audit documentation, tracking historical deployment discrepancies, incident logs, and legacy release note references to improve team-wide release transparency.

#### Added
- Added operational documentation suite:
  - `docs/deployment-history.md`: Detailed record of historical deployment timestamps and discrepancies.
  - `docs/incident-log.md`: Registry of production outages caused by improper tagging (Incidents 1, 2, and 3).
  - `docs/release-notes-old.md`: Archival record of legacy unstandardized release notes.
- Added structured sections to `CHANGELOG.md`.

#### Changed
- Updated documentation links and release guidance across team manuals.

#### Fixed
- Addressed total lack of incident traceability by providing a centralized incident record.

### Operational Guidance & Rollback Plan
- **Rollback Procedure**: If documentation references conflict with internal runbooks:
  ```bash
  git checkout v1.0.0
  ```

---

## Release v1.0.0

- **Release Date**: 2026-03-15
- **Git Tag**: [`v1.0.0`](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill) (Annotated)
- **Target Commit**: `ff1fb9a`
- **Deployment Target**: Initial Production Baseline
- **Rollback Target**: None (Initial release baseline)

### Summary of Changes
Initial production baseline release establishing the foundational Checkout Service repository structure, initial execution runner, and core documentation.

#### Added
- Initial Checkout Service implementation skeleton in `src/app.js`.
- Component documentation in `src/README.md`.
- Base assignment and operational overview in `README.md`.
- Initial `CHANGELOG.md` file.

#### Changed
- Baseline creation.

#### Fixed
- Initial repository setup.

### Operational Guidance & Rollback Plan
- **Rollback Procedure**: As this is the foundational baseline release, there is no previous Git release target. Rollback entails reverting the deployment to maintenance mode or tearing down the deployment pod.
