# Release-to-Deployment Traceability Map

This document establishes the single source of truth connecting Git tags, commit hashes, release dates, and production deployment records. It answers two critical operational questions during normal operations and outages:
1. **"What version and commit are currently running in production?"**
2. **"What is the exact, verified version and commit to roll back to if an incident occurs?"**

---

## 1. Structured Release Mapping Table

| Version (tag) | Commit (short hash) | Date | What Shipped | Environment & Deployment Status |
| :--- | :--- | :--- | :--- | :--- |
| **`v1.1.1`** | `82535d9` | 2026-05-04 | Standardized service runtime constants (`SERVICE_NAME`, `SERVICE_VERSION`), implemented clean entrypoint function `run()`, and added structured startup logging. | **Production (CURRENT ACTIVE)** |
| **`v1.1.0`** | `86989b6` | 2026-04-10 | Added operational documentation suite: historical deployment logs, incident register, and centralized legacy release notes. | **Staging / Stored Rollback Target (KNOWN-GOOD)** |
| **`v1.0.0`** | `ff1fb9a` | 2026-03-15 | Initial baseline release: scaffolding of checkout service structure, initial application runner, and repository documentation skeleton. | **Archive / Baseline Foundation** |

---

## 2. Operational Answers

### What is currently running in production?
- **Active Release Tag**: `v1.1.1`
- **Target Commit**: `82535d9`
- **Verification**: Inspecting the running service displays `SERVICE_NAME = checkout-service` and `SERVICE_VERSION = 0.0.0-placeholder`.

### What do I roll back to if `v1.1.1` fails?
- **Immediate Rollback Target**: `v1.1.0`
- **Target Commit**: `86989b6`
- **Status**: Verified known-good release prior to `v1.1.1`.
- **Secondary / Baseline Rollback Target**: `v1.0.0` (commit `ff1fb9a`).

---

## 3. Traceable Rollback Demonstration

When a production incident occurs (e.g., payment failure or 502 Bad Gateway), the operations team executes the following deterministic recovery sequence:

### Step 1: List all releases in strict descending semantic order
```bash
git tag --sort=-v:refname
```
*Expected Output:*
```text
v1.1.1   <-- Current release running in production
v1.1.0   <-- Immediately preceding known-good release target
v1.0.0   <-- Initial baseline release
```

### Step 2: Checkout the last known-good tag and redeploy
```bash
# Roll back working directory or deployment pipeline to the last known-good tag:
git checkout v1.1.0

# Verify the exact commit and release annotation:
git show --quiet v1.1.0

# Trigger the deployment / restart of the verified artifact:
npm start  # or deploy container built from tag v1.1.0
```

---

## 4. Why This Eliminates Rollback Guesswork

Our new, consistent semantic tagging convention (`vMAJOR.MINOR.PATCH`) transforms rollback from hazardous guesswork into a deterministic, reproducible operational procedure. In the legacy repository history, engineers faced ambiguous labels (`latest-good`, `release_2`, `v2-final-FINAL`) where sorting was corrupted and rollback targets pointed to broken, unsupported commits. Under the standardized convention, `git tag --sort=-v:refname` guarantees that the previous known-good release (`v1.1.0`) is always immediately identifiable, immutable, and backed by a full Git annotation audit trail.
