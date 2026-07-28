## Why

`spring-data-commons-2.7` currently targets `2.7.18-nes.patch.1` but does not yet have a complete, immutable, and independently auditable RELEASE lifecycle. The release must survive session loss, preserve unrelated work, and publish metadata that references only approved internal RELEASE dependencies.

## What Changes

- Freeze and inventory the `spring-data-commons-2.7` worktree before release edits.
- Create RELEASE metadata for `2.7.18-nes.patch.1` and replace all internal SNAPSHOT references with approved RELEASE versions.
- Audit and reuse qualifying development build/test evidence under the approved release-only-diff exception, then run local publication/effective-POM scan and consumer verification.
- Form a dedicated release commit containing component release documentation and no unrelated changes.
- Verify every discovered target GAV is absent from Nexus RELEASE before the coordinator-authorized deploy.
- Download and verify published POM/binary assets and checksums from Nexus, then run RELEASE-only consumption.
- Create annotated tag `v2.7.18-nes.patch.1` on the exact release commit only after Nexus verification.
- Record Git, Nexus, documentation, and manifest evidence before archiving this change.

## Capabilities

### New Capabilities

- `component-release-spring-data-commons-2.7`: Release `spring-data-commons-2.7` `2.7.18-nes.patch.1` with reproducible local gates, immutable Nexus publication, exact Git tagging, documentation, and archive evidence.

### Modified Capabilities

- `gav-renaming`: Promote the component and Framework BOM versions from development SNAPSHOTs to the approved RELEASEs.
- `nexus-config`: Route the target version to the immutable Nexus RELEASE repository instead of snapshots.

## Impact

- **Repository:** `spring-data-commons-2.7`
- **Release version:** `2.7.18-nes.patch.1`
- **Representative GAVs:** `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:2.7.18-nes.patch.1`
- **Internal RELEASE dependencies:** `spring-framework-5.3`
- **Explicit exclusions:** none
- **Release documentation:** `README.adoc`, `doc/QUICK_START.md`, `doc/USER_MANUAL.md`, `doc/GAV_MAPPING.md`, `doc/REQUIREMENTS.md`, `doc/VULNERABILITY_REPORT.md`
- **External systems:** Nexus RELEASE and the repository's configured `origin`
- **Credentials:** Remain exclusively in user-level Gradle/Maven configuration and are never copied into this change or Git
- **Approved verification exception:** Do not repeat `make build` or `make test` when the release-only diff contains no production/test source changes; the coordinating main session must still run a serialized local install before deploy, using the recorded no-clean Maven invocation for this release
