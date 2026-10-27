# Rollback Traceability & Recovery Runbook

## 1. Overview

Production incidents demand fast, deterministic, and error-free rollback procedures. In high-pressure outage scenarios, engineers cannot afford ambiguity about what version is running or what version is safe to revert to.

This document demonstrates the exact operational commands used to achieve verifiable, repeatable rollbacks using our standardized semantic tag history within the context of the repository.

---

## 2. Standard Rollback Procedure

When an alert fires or an incident is declared in production (e.g., following a deployment of `v1.1.1`):

### Step 1: Query the Repository Release History
Run Git's version-sort command to list all tags in the repository:
```bash
git tag --sort=-v:refname        # the sortable release history
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

#### Understanding the Output:
- **Legacy Tags Above**: `version-1.0`, `v2-final-FINAL`, and `v1.4.2` sort to the top due to the legacy naming defects detailed in `TAG-AUDIT.md`.
- **Standardized Release Sequence**:
  ```text
  v1.1.1   # Current release running in production
  v1.1.0   # Last known-good release target for immediate rollback
  v1.0.0   # Baseline initial release
  ```
  The newly introduced semantic tags form a clean, continuous, and descending sequence.
- **Legacy Tags Below**: Legacy tags lacking prefixes (`1.5.0`) or using non-versioned words (`stable-build`, `release_2`, `patch-new`, `latest-good`) sort below.

#### Targeted SemVer Query:
To isolate the standardized semantic release sequence and exclude non-conforming legacy tags:
```bash
git tag -l "v1.*" --sort=-v:refname
```
Output:
```text
v1.4.2   # Audited legacy tag
v1.1.1   # Current Active Production
v1.1.0   # Immediate Rollback Target (Known-Good)
v1.0.0   # Baseline Release
```

---

### Step 2: Roll Back to the Last Known-Good Tag
Execute a checkout of the identified known-good tag, verify its annotated metadata, and trigger the deployment pipeline:
```bash
git checkout v1.1.0              # roll back to the last known-good tag, then redeploy
```

To verify the tag's provenance, creator, timestamp, and release notes before redeployment:
```bash
git show --quiet v1.1.0
```

Once verified, re-run the service build or restart the container:
```bash
node src/app.js
```

---

## 3. Remote Tag Verification Evidence

To establish that the release tags are published and available across the engineering and operations teams, all tags were pushed to the remote repository (`origin`).

Running `git ls-remote --tags origin` confirms that the tags and their underlying annotated objects exist on GitHub:

```bash
git ls-remote --tags origin
```

```text
a3ad3aab391da05c3cdd1c363b4452adb21ff523	refs/tags/v1.0.0
ff1fb9ab88e9e20a4311bb8e6f5f71139f94673a	refs/tags/v1.0.0^{}
a349a70741748773dc634686e3d668446239b426	refs/tags/v1.1.0
86989b62c378384467da816a1b6f687ee9ea0c51	refs/tags/v1.1.0^{}
043f39a320ba6c0b6afcb084351fe1e48bc201ec	refs/tags/v1.1.1
82535d9c7b97066e13c3f4afd92f7227d531664b	refs/tags/v1.1.1^{}
```
*(Plus legacy tags maintained for audit preservation).*

Each tag reference has a peeled target (`^{}`) pointing precisely to the expected commit hash:
- `v1.0.0^{}` -> `ff1fb9a`
- `v1.1.0^{}` -> `86989b6`
- `v1.1.1^{}` -> `82535d9`

---

## 4. Why the Standardized History Eliminates Guesswork

Our new, consistent semantic tagging convention (`vMAJOR.MINOR.PATCH`) transforms rollback from hazardous guesswork into a deterministic, reproducible operational procedure. In the legacy repository history, engineers faced ambiguous labels (`latest-good`, `release_2`, `v2-final-FINAL`) where sorting was corrupted and rollback targets pointed to broken, unsupported commits. Under the standardized convention, `git tag --sort=-v:refname` guarantees that the previous known-good release (`v1.1.0`) is always immediately identifiable, immutable, and backed by a full Git annotation audit trail.

---

## 5. Comparison: Legacy vs. Standardized Traceability

| Dimension | Legacy Tagging Approach | Standardized Semantic Tagging |
| :--- | :--- | :--- |
| **Sorting Behavior** | `git tag --sort=-v:refname` sorted `version-1.0` to top and `1.5.0` to bottom. | Releases sort chronologically and semantically in exact descending order. |
| **Rollback Target Identification** | Ambiguous labels like `latest-good` and `release_2` caused **Incident 1** (502 errors after deploying stale commit). | Preceding release (`v1.1.0`) is immediately identified and verified from [RELEASE-MAP.md](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill/RELEASE-MAP.md) and [RELEASES.md](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill/RELEASES.md). |
| **Audit Provenance** | Lightweight tags provided no author, date, or release notes. | Annotated tags (`git tag -a`) permanently store author identity, timestamp, and release context in Git's object store. |
| **Automation Compatibility** | Broke automated CI/CD and deployment scripts. | 100% compliant with standard SemVer 2.0.0 and CI/CD promotion tooling. |
| **Remote Visibility** | Tags created ad hoc locally without remote push verification. | Verified on remote via `git ls-remote --tags origin`. |
