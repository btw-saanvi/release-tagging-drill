# Checkout Service Release & Versioning Convention

This document defines the official engineering policy for versioning and tagging software releases across the Checkout Service. All engineers, release managers, and CI/CD pipelines must strictly adhere to these standards.

---

## 1. Semantic Versioning Rules (SemVer 2.0.0)

All releases follow the Semantic Versioning specification format: `MAJOR.MINOR.PATCH`. Version numbers increment based on the nature of the changes introduced in the release:

### 1. MAJOR Version Bump
- **Definition**: Incremented when backward-incompatible changes, breaking API modifications, breaking database schema shifts, or architectural rewrites are introduced that require consumers to modify their integration.
- **Rules**: Reset MINOR and PATCH to `0`.
- **Concrete Example**:
  - `v1.4.2 -> v2.0.0`
  - *Context*: The checkout payment endpoint contract is restructured from synchronous token exchange to an asynchronous webhook-based authorization model, breaking existing client integrations.

### 2. MINOR Version Bump
- **Definition**: Incremented when backward-compatible new functionality, new API endpoints, or substantial non-breaking features are added to the service.
- **Rules**: Reset PATCH to `0`.
- **Concrete Example**:
  - `v1.4.2 -> v1.5.0`
  - *Context*: Introducing a new optional Redis-based payment token caching feature and adding an audit logging hook without modifying existing endpoint behaviors.

### 3. PATCH Version Bump
- **Definition**: Incremented when backward-compatible bug fixes, security patches, internal refactorings, or performance optimizations are introduced that do not add new public capabilities or break contracts.
- **Rules**: Only the PATCH number increments.
- **Concrete Example**:
  - `v1.4.2 -> v1.4.3`
  - *Context*: Resolving a socket connection timeout when communicating with the third-party payment gateway during peak traffic load.

---

## 2. Tag Naming Format

Every release tag in the repository must adhere to the standardized prefix and numbering convention:

### Format Specification
```text
vMAJOR.MINOR.PATCH
```

### Formatting Rules:
1. **Mandatory Prefix**: The tag MUST begin with a lowercase `v`.
2. **Three-Segment Numbers**: The version MUST contain exactly three non-negative integers separated by dots (`MAJOR.MINOR.PATCH`).
3. **No Suffixes for Final Releases**: Do not append suffixes such as `-final`, `-FINAL`, or `-prod`.
4. **No Arbitrary Words or Characters**: Never use underscore separators (e.g., `release_2`), descriptive words (`stable-build`, `patch-new`), or expanded prefixes (`version-1.0`).
5. **Concrete Example**:
   ```text
   v1.0.0
   v1.1.0
   v1.1.1
   ```

---

## 3. Annotated vs. Lightweight Tags Policy

### Official Policy: Annotated Tags for All Production and Staging Releases
The engineering team strictly mandates the use of **annotated tags** for all release milestones. **Lightweight tags are prohibited** for official releases and may only be used for temporary, local developer bookmarks.

### Rationale: Why Annotated Tags?
1. **Full Provenance & Audit Trail**: An annotated tag is stored as an independent, immutable object in the Git object database (`type: tag`). It permanently records:
   - The tagger’s real name and verified email.
   - The exact date and timestamp of tag creation.
   - A descriptive release summary explaining what shipped.
   - Cryptographic GPG/SSH signatures (when using `-s`), guaranteeing tamper-proof authenticity.
2. **Protection Against Accidental Overwrites**: Unlike lightweight tags (which are raw pointers indistinguishable from local branches), annotated tags are explicitly treated as immutable releases by Git tooling and release management platforms.
3. **Automated Release Notes & CI/CD Extraction**: Tools such as `git describe`, changelog generators, and deployment scripts extract the annotated message directly to verify deployment contents.

### Exact Command to Create Release Tags
```bash
git tag -a vMAJOR.MINOR.PATCH -m "Release MAJOR.MINOR.PATCH: <concise summary of what changed>"
```

#### Real-world Example:
```bash
git tag -a v1.1.0 -m "Release 1.1.0: Add structured checkout service logging and deployment tracking docs"
```

---

## 4. Pre-Release Tagging Rules

To validate candidate builds before full production promotion, pre-release tags must follow a structured, sortable schema.

### Pre-Release Format Specification
```text
vMAJOR.MINOR.PATCH-<stage>.<number>
```
Where:
- `<stage>` is one of: `alpha`, `beta`, or `rc` (Release Candidate).
- `<number>` is an incrementing non-negative integer starting at `1`.

### Concrete Examples:
- Alpha build: `v1.5.0-alpha.1`
- Beta build: `v1.5.0-beta.1`
- Release candidates: `v1.5.0-rc.1`, `v1.5.0-rc.2`

### Precedence and Ordering Relative to Final Release
Under the SemVer 2.0.0 specification (Item 11) and Git version-sort mechanics (`git tag --sort=v:refname`), a pre-release version has **lower precedence** than the associated normal version:

```text
v1.5.0-alpha.1  <  v1.5.0-beta.1  <  v1.5.0-rc.1  <  v1.5.0-rc.2  <  v1.5.0
```

When sorting in descending order with `git tag --sort=-v:refname`:
```text
v1.5.0          # The official production release sorts FIRST
v1.5.0-rc.2     # Pre-release candidates sort BELOW final release
v1.5.0-rc.1
v1.5.0-beta.1
v1.5.0-alpha.1
```

This guarantees that automated CI/CD deployment pipelines selecting the topmost tag will reliably pick the final, stabilized release `v1.5.0` and will never mistakenly promote a pre-release candidate into production over a final build.
