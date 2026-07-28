# Static RELEASE preparation evidence

Recorded at: `2026-07-27T10:47:29Z`

## Ownership and repository baseline

- Change: `release-2-7-18-nes-patch-1` (`spec-driven`, apply-ready)
- Repository: `spring-data-commons-2.7`
- Branch: `2.7.x-bjca-patch`
- Baseline HEAD: `44eebbfeec112dcb47719be1a295eb225a8d657e`
- `origin` fetch/push: `https://github.com/atbjca/spring-data-commons.git`
- Additional remote fetch/push: `git@192.168.131.1:NES/spring-data-commons.git`
- Central lease: owner `wave4-commons-27`, acquired `2026-07-27T10:45:14Z`, expires `2026-07-27T16:45:14Z`
- Central lease commit: `41ba5f18d0fa0014ee94a3dfaac4456fbbd89f96`
- Initial tracked changes: none
- Initial untracked paths: `.claude/`, `openspec/changes/release-2-7-18-nes-patch-1/`
- Excluded local/generated paths: `.claude/`, `target/`
- Active component change: `release-2-7-18-nes-patch-1`

No credential values were read or recorded. `.claude/` and `target/` are not release inputs and must not be staged.

## Toolchain and target

- Runtime JDK: Amazon Corretto `17.0.17`
- Maven wrapper distribution: Maven `3.9.5`
- Project bytecode target: Java 8
- Original component version: `2.7.18-nes.patch.1-SNAPSHOT`
- RELEASE version: `2.7.18-nes.patch.1`
- Representative GAV: `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:jar:2.7.18-nes.patch.1`
- Explicit exclusions: none

## Dependency gate

The central catalog declares only `spring-framework-5.3` as an internal upstream. Its central manifest state is `tagged`, which follows `nexus-verified`, and records:

- version `5.3.39-nes.patch.1`
- release commit `413bca94d5a1190bbb0950164e0cfdabb9cd0a57`
- tag `v5.3.39-nes.patch.1`
- Nexus POM and JAR URLs
- SHA-256 checksum evidence
- RELEASE-only consumer evidence

The component POM now imports `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1`; all nine direct internal Framework dependencies are version-managed by that RELEASE BOM. The parent `org.springframework.data.build:spring-data-parent:2.7.18` is an external RELEASE.

## Approved build/test exception

Committed development evidence is reused for this RELEASE:

- `34fb3696` recorded `./mvnw clean test`: 3328 tests, 0 failures, 0 errors.
- `cce2b031` switched direct Spring dependencies to the NES Framework BOM and its archived OpenSpec records a later `./mvnw clean test` with 0 failures and 0 errors.
- Existing Surefire XML evidence contains 245 suites, 3328 tests, 0 failures, 0 errors, and 7 skipped tests, last written on 2026-07-09 after the dependency change.
- `git diff cce2b031..44eebbfe` contains no production or test source changes.
- The current RELEASE preparation is restricted to version/dependency metadata, release documentation, OpenSpec, and evidence.

Therefore `make build` and `make test` are explicitly not repeated. Any production or test source change before the release commit invalidates this exception.

## Pre-install publication hypothesis

Static Maven inspection shows one project with default `jar` packaging and no modules. The parent attaches a sources JAR, and the earlier SNAPSHOT deployment recorded POM, main JAR, sources JAR, and Maven metadata. The expected publication therefore has one GAV with these versioned assets:

- `bjca-footstone-bpring-data-commons-2.7.18-nes.patch.1.pom`
- `bjca-footstone-bpring-data-commons-2.7.18-nes.patch.1.jar`
- `bjca-footstone-bpring-data-commons-2.7.18-nes.patch.1-sources.jar`

This is a static hypothesis, not the final publication inventory. The coordinating main session must run the following command serially before any release commit or deploy:

```bash
make install
```

`make install` expands to `./mvnw -DskipTests clean install`. After it succeeds, inventory all installed/generated assets, scan every generated POM for internal `cn.bjca.footstone` SNAPSHOT references, and run the RELEASE-only consumer test.

The coordinating main session later completed an approved no-clean incremental install instead. The confirmed publication inventory, scan, and consumer results are recorded in `evidence/local-release.md`.
