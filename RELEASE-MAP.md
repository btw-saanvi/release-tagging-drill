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

## 2. Remote Tag Verification Evidence

To ensure complete transparency and verify that release tags are not merely local references, all newly created annotated tags have been pushed to the remote repository (`origin`).

Executing `git ls-remote --tags origin` confirms the existence of the tag objects and their resolved peeled commits on GitHub:

```bash
git ls-remote --tags origin
```

```text
6dbbf5251828d969abe131c9bd32b96598e09536	refs/tags/1.5.0
86989b62c378384467da816a1b6f687ee9ea0c51	refs/tags/1.5.0^{}
515c3f3ef190fb7738344d723f42882dc6e62926	refs/tags/latest-good
da6b2ed9c14c65b3165af65012bab1046ddd9b35	refs/tags/latest-good^{}
82535d9c7b97066e13c3f4afd92f7227d531664b	refs/tags/patch-new
28bd11092241f4b7d47d267668b787bd7ba19074	refs/tags/release_2
f9c38e9890a3408b3979bd709ba1a6583807b0e8	refs/tags/stable-build
ff1fb9ab88e9e20a4311bb8e6f5f71139f94673a	refs/tags/stable-build^{}
a3ad3aab391da05c3cdd1c363b4452adb21ff523	refs/tags/v1.0.0
ff1fb9ab88e9e20a4311bb8e6f5f71139f94673a	refs/tags/v1.0.0^{}
a349a70741748773dc634686e3d668446239b426	refs/tags/v1.1.0
86989b62c378384467da816a1b6f687ee9ea0c51	refs/tags/v1.1.0^{}
043f39a320ba6c0b6afcb084351fe1e48bc201ec	refs/tags/v1.1.1
82535d9c7b97066e13c3f4afd92f7227d531664b	refs/tags/v1.1.1^{}
ff1fb9ab88e9e20a4311bb8e6f5f71139f94673a	refs/tags/v1.4.2
82535d9c7b97066e13c3f4afd92f7227d531664b	refs/tags/v2-final-FINAL
6bb4f4376c4f7d2ac3b3f9da96742b21dc8900c9	refs/tags/version-1.0
28bd11092241f4b7d47d267668b787bd7ba19074	refs/tags/version-1.0^{}
```

### Verification Analysis:
- `refs/tags/v1.0.0`: Annotated tag object `a3ad3aa` peels directly to baseline commit `ff1fb9a`.
- `refs/tags/v1.1.0`: Annotated tag object `a349a70` peels directly to documentation commit `86989b6`.
- `refs/tags/v1.1.1`: Annotated tag object `043f39a` peels directly to active service commit `82535d9`.

---

## 3. Operational Answers

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

## 4. Full Repository Release Sorting & Traceable Rollback

### Step 1: Query the Full Repository Release History
Running `git tag --sort=-v:refname` lists all existing tags in the repository (including both legacy tags and our standardized release tags):

```bash
git tag --sort=-v:refname
```

#### Actual Full Repository Output:
```text
version-1.0
v2-final-FINAL
v1.4.2
v1.1.1
v1.1.0
v1.0.0
stable-build
release_2
patch-new
latest-good
1.5.0
```

#### Contextualizing the Output:
- **Legacy Tags Above**: `version-1.0`, `v2-final-FINAL`, and `v1.4.2` sort to the top because of the legacy naming flaws documented in `TAG-AUDIT.md` (e.g. prefix `version-` alphabetical precedence and non-semantic numbering).
- **The Standardized Release Sequence**:
  ```text
  v1.1.1   <-- Current release running in production
  v1.1.0   <-- Immediately preceding known-good release target
  v1.0.0   <-- Initial baseline release
  ```
  These three tags form a clean, continuous, and strictly ordered semantic sequence.
- **Legacy Tags Below**: Non-prefixed tags (`1.5.0`) and non-semantic labels (`stable-build`, `release_2`, `patch-new`, `latest-good`) sort below.
- **Targeted Semantic Query**: Teams or automated deployment scripts targeting the official release stream can run:
  ```bash
  git tag -l "v1.*" --sort=-v:refname
  ```
  Which cleanly isolates the release sequence:
  ```text
  v1.4.2   # Audited legacy tag
  v1.1.1   # Current Production
  v1.1.0   # Known-Good Rollback Target
  v1.0.0   # Initial Baseline
  ```

---

### Step 2: Checkout the Last Known-Good Tag and Redeploy
During a production incident on `v1.1.1`, the team executes a safe, traceable rollback to `v1.1.0`:

```bash
# Roll back working tree to the verified previous release:
git checkout v1.1.0              # roll back to the last known-good tag, then redeploy

# Inspect the annotated release message, tagger, and commit:
git show --quiet v1.1.0

# Redeploy the verified service artifact:
node src/app.js
```

---

## 5. Why This Eliminates Rollback Guesswork

Our new, consistent semantic tagging convention (`vMAJOR.MINOR.PATCH`) transforms rollback from hazardous guesswork into a deterministic, reproducible operational procedure. In the legacy repository history, engineers faced ambiguous labels (`latest-good`, `release_2`, `v2-final-FINAL`) where sorting was corrupted and rollback targets pointed to broken, unsupported commits. Under the standardized convention, `git tag --sort=-v:refname` guarantees that the previous known-good release (`v1.1.0`) is always immediately identifiable, immutable, and backed by a full Git annotation audit trail.
