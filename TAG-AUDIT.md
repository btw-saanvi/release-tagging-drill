# Release Tag Audit Report

## 1. Overview and Objectives

This audit evaluates the historical Git tagging practices in the Checkout Service repository. Prior to this audit, multiple developers created tags ad hoc without standard conventions, mixing lightweight tags, mutable labels, duplicate pointers, and non-semantic names. This lack of rigor caused production outages, misidentified rollback targets, and untraceable deployments (as documented in `docs/incident-log.md` and `docs/deployment-history.md`).

---

## 2. Baseline Audit Evidence

The repository history and existing tags were evaluated by executing three core Git commands:

### Command 1: `git tag` (Alphabetical Listing)
```text
1.5.0
latest-good
patch-new
release_2
stable-build
v1.4.2
v2-final-FINAL
version-1.0
```

### Command 2: `git log --oneline --decorate --all` (Commit Association)
```text
82535d9 (HEAD -> feature/release-tagging, tag: v2-final-FINAL, tag: patch-new, origin/main, origin/HEAD, main) feat: add dummy checkout service
86989b6 (tag: 1.5.0) docs: add old release notes and changelog
da6b2ed (tag: latest-good) docs: add broken deployment history and incident log
28bd110 (tag: version-1.0, tag: release_2) docs: add broken deployment history and incident log
ff1fb9a (tag: v1.4.2, tag: stable-build) docs: scaffold assignment repository
c17d4c4 feat: add dummy checkout service
9426db9 docs: add inconsistent old release notes and changelog
4febcc6 first commit
```

### Command 3: `git tag --sort=-v:refname` (Version-Sort Evaluation)
```text
version-1.0
v2-final-FINAL
v1.4.2
stable-build
release_2
patch-new
latest-good
1.5.0
```

---

## 3. Detailed Audit Findings: Five Specific Tagging Problems

Below are specific tagging problems identified in the repository, citing the exact tag evidence, impact on team workflow, and specific rollback and traceability risks.

---

### Problem 1: Inconsistent Prefixing and Inverted Version Sorting
- **Evidence**: `version-1.0`, `1.5.0`, and `v1.4.2`
- **What it Means for the Team**: 
  The team lacks a single uniform naming standard. Three releases used three completely different conventions: one used a word prefix with a hyphen (`version-1.0`), another completely omitted any prefix (`1.5.0`), and another used a single lowercase `v` (`v1.4.2`).
- **Rollback and Traceability Risk**:
  When automated deployment scripts or site reliability engineers (SREs) sort tags using Git's standard version sorting command `git tag --sort=-v:refname`, Git parses the string alphabetically when prefixes vary. Because `"version-"` precedes `"v"` alphabetically, Git ranks `version-1.0` as the highest version in the entire repository, while `1.5.0` (lacking any letter prefix) sorts to the very bottom. Automated CI/CD CD pipeline triggers that pick the "latest" release would deploy `version-1.0` over `1.5.0` and `v2-final-FINAL`, causing a catastrophic production regression.

---

### Problem 2: Subjective, Emotional Naming and Contradictory Duplicate Tags on a Single Commit
- **Evidence**: `v2-final-FINAL` and `patch-new` (both pointing to commit `82535d9`)
- **What it Means for the Team**:
  Developers assigned informal, emotional file-like labels (`v2-final-FINAL`) and generic descriptions (`patch-new`) to the exact same commit (`82535d9`). In `docs/release-notes-old.md`, the author even wrote: *"production-ready? maybe"*. There is no clarity on whether this commit represents a breaking major upgrade (v2.0.0) or a minor maintenance patch.
- **Rollback and Traceability Risk**:
  During a deployment emergency, an engineer cannot determine whether deploying or rolling back to `82535d9` will execute breaking schema/API migrations (implied by `v2`) or safe patch fixes (implied by `patch-new`). If an incident occurs in production, operators cannot trace dependencies or verify test validation logs.

---

### Problem 3: Conflicting Dual Tags with Underscores and Zero Semantic Hierarchy
- **Evidence**: `release_2` and `version-1.0` (both pointing to commit `28bd110`)
- **What it Means for the Team**:
  Commit `28bd110` is tagged simultaneously as `release_2` (using an underscore and no minor/patch numbers) and `version-1.0` (an annotated tag). The team has two competing version numbers for the exact same snapshot in history.
- **Rollback and Traceability Risk**:
  This directly triggered **Incident 1** (`docs/incident-log.md`, 2026-04-22). Production suffered `502 Bad Gateway` errors, and the operations team initiated an emergency rollback to `release_2` under the belief it was a stable Release 2 artifact. Instead, `release_2` pointed to an unsupported commit that lacked critical payment gateway integrations, worsening the downtime. Because `release_2` was a lightweight tag with no metadata, there was no record of what commit was actually verified.

---

### Problem 4: Mutable State Used as a Static Release Tag
- **Evidence**: `latest-good` (annotated tag pointing to commit `da6b2ed`)
- **What it Means for the Team**:
  A team member attempted to create a tag representing a moving status ("latest known-good build"). Tags in Git are static, immutable pointers, whereas mutable concepts belong on branches or deployment registries. Making matters worse, `latest-good` points to commit `da6b2ed`, which is older than `28bd110`, `86989b6`, and `82535d9`.
- **Rollback and Traceability Risk**:
  This caused **Incident 2** (`docs/incident-log.md`, 2026-05-04). A hotfix was deployed directly from `latest-good`. Because `latest-good` does not correspond to any semantic version or changelog entry, the team had no record of what was running in production. When an incident occurred, responders could not determine what changes had shipped or what version preceded it.

---

### Problem 5: Non-Semantic, Ambiguous "Branch-like" Tag Names
- **Evidence**: `stable-build` (annotated tag pointing to commit `ff1fb9a`, sharing commit with `v1.4.2`)
- **What it Means for the Team**:
  The tag name `stable-build` provides no version number, no scope of changes, and no date. It duplicates commit `ff1fb9a`, which is also tagged `v1.4.2`.
- **Rollback and Traceability Risk**:
  This caused **Incident 3** (`docs/incident-log.md`, 2026-03-31), where operations logged: *"Unknown version currently running in production"*. During an outage, responders could not match running artifacts against release documentation because tags like `stable-build` cannot be sorted or mapped to release tickets. If a rollback is needed, an engineer cannot tell if `stable-build` is newer or older than other deployed artifacts.

---

### Problem 6: Chronological Inversion of Version Numbers Across Git History
- **Evidence**: `v1.4.2` on commit `ff1fb9a` versus `version-1.0` on commit `28bd110`
- **What it Means for the Team**:
  The version progression across the Git history is completely backwards. Commit `ff1fb9a` is the earliest scaffold commit, yet it is tagged as `v1.4.2`. Later in history, commit `28bd110` is tagged as `version-1.0`. In addition, `docs/release-notes-old.md` claims `v1.4.2` added payment token caching and timeout fixes, but git diff inspection shows commit `ff1fb9a` only contains markdown documentation and a scaffold placeholder.
- **Rollback and Traceability Risk**:
  Automated bisecting, rollback systems, and audit tools that inspect linear Git history will assume `ff1fb9a` represents a mature release (`v1.4.2`) containing token caching. Rolling back to `v1.4.2` actually deploys unfinished skeleton code, guaranteeing an immediate production outage.

---

### Problem 7: Reliance on Lightweight Tags for Production Releases
- **Evidence**: `release_2`, `patch-new`, `v1.4.2`, and `v2-final-FINAL` (all lightweight tags)
- **What it Means for the Team**:
  Developers used lightweight tags (`git tag <name>`) instead of annotated tags (`git tag -a <name> -m <msg>`). A lightweight tag creates no object in `.git/objects`; it is simply a raw commit pointer that does not store the author's identity, email, creation timestamp, or release message.
- **Rollback and Traceability Risk**:
  There is zero cryptographic or audit traceability. Anyone can move or delete a lightweight tag without leaving an audit trail in the tag object database. During root-cause analysis after a failed deployment, auditors cannot determine who authorized the tag or what verification was performed prior to deployment.

---

## 4. Legacy Tag Inventory Summary Table

| Tag Name | Tag Type | Target Commit | Issues Identified | Incident Association |
| :--- | :--- | :--- | :--- | :--- |
| `version-1.0` | Annotated | `28bd110` | Inconsistent prefix (`version-`), breaks `git tag --sort=-v:refname` | Incident 1 |
| `release_2` | Lightweight | `28bd110` | Non-semver, underscore syntax, duplicate tag on commit `28bd110` | Incident 1 (Failed rollback) |
| `latest-good` | Annotated | `da6b2ed` | Mutable concept as static tag, points to stale older commit | Incident 2 (Undocumented release) |
| `stable-build` | Annotated | `ff1fb9a` | Non-semantic name, duplicate tag on commit `ff1fb9a` | Incident 3 (Outage confusion) |
| `v1.4.2` | Lightweight | `ff1fb9a` | Inverted version history, lightweight tag, false release notes | Incident 3 |
| `1.5.0` | Annotated | `86989b6` | Missing `v` prefix, sorts to bottom of release list | General sorting failure |
| `patch-new` | Lightweight | `82535d9` | Non-versioned descriptor, duplicate tag on commit `82535d9` | Incident 3 |
| `v2-final-FINAL` | Lightweight | `82535d9` | Subjective colloquial naming, lightweight tag, ambiguous scope | Incident 3 |

---

## 5. Conclusion & Action Items

The audit conclusively proves that arbitrary tag naming and reliance on lightweight tags crippled deployment traceability and caused multiple production outages. To eliminate these risks, the team must:
1. Adopt a strict Semantic Versioning (`SemVer 2.0.0`) policy with a mandatory `v` prefix (`vMAJOR.MINOR.PATCH`).
2. Require annotated tags (`git tag -a`) for all releases.
3. Establish a transparent Release Map linking version tags to specific commits and deployment audit logs.
