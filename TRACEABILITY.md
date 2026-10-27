# Rollback Traceability & Recovery Runbook

## 1. Overview

Production incidents demand fast, deterministic, and error-free rollback procedures. In high-pressure outage scenarios, engineers cannot afford ambiguity about what version is running or what version is safe to revert to.

This document demonstrates the exact operational commands used to achieve verifiable, repeatable rollbacks using our standardized semantic tag history.

---

## 2. Standard Rollback Procedure

When an alert fires or an incident is declared in production (e.g., following a deployment of `v1.1.1`):

### Step 1: Query the Sortable Release History
Run Git's version-sort command to list all releases in strict descending semantic order:
```bash
git tag --sort=-v:refname        # the sortable release history
```

**Output Inspection:**
```text
v1.1.1   # Current release running in production
v1.1.0   # Last known-good release target for immediate rollback
v1.0.0   # Baseline initial release
```

Because all tags follow the strict `vMAJOR.MINOR.PATCH` pattern, Git's version sort places the immediately previous release (`v1.1.0`) directly below the current release (`v1.1.1`).

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

Once verified, re-run the service build or trigger your container deployment pipeline using tag `v1.1.0`.

---

## 3. Why the Standardized History Eliminates Guesswork

Our new, consistent semantic tagging convention (`vMAJOR.MINOR.PATCH`) transforms rollback from hazardous guesswork into a deterministic, reproducible operational procedure. In the legacy repository history, engineers faced ambiguous labels (`latest-good`, `release_2`, `v2-final-FINAL`) where sorting was corrupted and rollback targets pointed to broken, unsupported commits. Under the standardized convention, `git tag --sort=-v:refname` guarantees that the previous known-good release (`v1.1.0`) is always immediately identifiable, immutable, and backed by a full Git annotation audit trail.

---

## 4. Comparison: Legacy vs. Standardized Traceability

| Dimension | Legacy Tagging Approach | Standardized Semantic Tagging |
| :--- | :--- | :--- |
| **Sorting Behavior** | `git tag --sort=-v:refname` sorted `version-1.0` to top and `1.5.0` to bottom. | Releases sort chronologically and semantically in exact descending order. |
| **Rollback Target Identification** | Ambiguous labels like `latest-good` and `release_2` caused **Incident 1** (502 errors after deploying stale commit). | Preceding release (`v1.1.0`) is immediately identified and verified from [RELEASE-MAP.md](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill/RELEASE-MAP.md) and [RELEASES.md](file:///c:/Users/Saanvi%20Garg/Desktop/release-tagging-drill/RELEASES.md). |
| **Audit Provenance** | Lightweight tags provided no author, date, or release notes. | Annotated tags (`git tag -a`) permanently store author identity, timestamp, and release context in Git's object store. |
| **Automation Compatibility** | Broke automated CI/CD and deployment scripts. | 100% compliant with standard SemVer 2.0.0 and CI/CD promotion tooling. |
