# Static RELEASE preparation evidence

Recorded: 2026-07-27 (Asia/Shanghai)

## Ownership and repository

- Component: `spring-data-commons-3.5`.
- Change: `release-3-5-13-nes-patch-1` (`spec-driven`, apply-ready).
- Assigned owner lease: `wave4-commons-35`.
- Lease acquired: `2026-07-27T10:45:15Z`; expires: `2026-07-27T16:45:15Z`.
- Central lease commit: `41ba5f18d0fa0014ee94a3dfaac4456fbbd89f96`.
- Branch: `3.5.x-bjca-patch`.
- Preparation base HEAD: `1c50d9ef91df6dfa17749f453b40a022af849195`.
- `origin` fetch/push: `https://github.com/atbjca/spring-data-commons.git`.
- Additional `gitlab` fetch/push: `git@192.168.131.1:NES/spring-data-commons.git`.
- Active OpenSpec changes: only `release-3-5-13-nes-patch-1`.
- Toolchain inspected without invoking Maven: Amazon Corretto JDK `17.0.17`; Maven wrapper distribution `3.9.16`.
- Credentials remain in user-level Maven settings and were neither read nor recorded.

## Worktree inventory

Before RELEASE edits there were no tracked worktree changes. The only
non-ignored untracked content was this release OpenSpec directory. Ignored
local/generated paths were `.claude/`, `.mvn/.develocity/`, and `target/`.
They must remain excluded from release staging. Existing `target/` files are
stale SNAPSHOT build output and are not RELEASE evidence.

The approved release diff is limited to `pom.xml`, component documentation,
and this OpenSpec change. It contains no production source, test source, local
tool directory, or credential file.

## Version and dependency gate

- Component RELEASE: `3.5.13-nes.patch.1`.
- Internal upstream BOM: `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:6.2.19-nes.patch.1`.
- Approved publication exclusions: none.

The central manifest records `spring-framework-6.2` as `tagged` at release
commit `b2bb78eb3c6571aa17d1c1a52e6f258d4b768acb` with local annotated tag
`v6.2.19-nes.patch.1`. It also records Nexus RELEASE POM/JAR URLs, checksums,
and RELEASE-only consumer evidence. This satisfies the declared upstream gate;
Data Commons 3.5 has no other internal upstream component.

Representative upstream evidence URLs:

- `http://192.168.131.36:8088/repository/releases/cn/bjca/footstone/bpring/bjca-footstone-bpring-framework-bom/6.2.19-nes.patch.1/bjca-footstone-bpring-framework-bom-6.2.19-nes.patch.1.pom`
- `http://192.168.131.36:8088/repository/releases/cn/bjca/footstone/bpring/bjca-footstone-bpring-core/6.2.19-nes.patch.1/bjca-footstone-bpring-core-6.2.19-nes.patch.1.jar`

## Static publication assessment

The root POM declares no modules and no alternate packaging, so Maven's
default `jar` packaging applies. There are no approved or configured publication
exclusions. The statically inferred publication set is therefore one GAV:

- `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:jar:3.5.13-nes.patch.1`

Prior ignored `target/` output contained the SNAPSHOT main JAR and sources JAR.
That output predates the RELEASE edit and is not RELEASE evidence. The actual
RELEASE publication set and assets were subsequently confirmed by the local
installation recorded below.

## Approved build/test reuse exception

The operator explicitly directed this release run not to repeat `make build`
or `make test`. Existing component documentation records a successful full
`clean install` with 3,630 tests, zero failures, zero errors, and seven skipped,
plus the targeted CVE regression counts. Static release review finds no
production or test source change. On that basis, tasks 3.1 and 3.2 use the
operator-approved release-only diff exception.

This exception does not cover local publication. The main session held the
global build lock and ran this incremental command:

```bash
JAVA_HOME=/Users/anan/.sdkman/candidates/java/17.0.17-amzn \
JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8 \
MAVEN_OPTS='-Xmx4g -Dfile.encoding=UTF-8' \
./mvnw -DskipTests install
```

This deliberately omitted `clean`, `make build`, and `make test`.

## Local RELEASE publication and consumer verification

The coordinator-reported incremental Maven installation completed successfully
in `59.987s`. Maven executed 15 lifecycle goals. Test execution was skipped by
`-DskipTests`, while test sources were still compiled as part of the lifecycle.

The complete installed publication set contains one GAV:

- `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:jar:3.5.13-nes.patch.1`

Its Maven Local version directory contains exactly the publication metadata
and expected binary assets relevant to this project:

- `bjca-footstone-bpring-data-commons-3.5.13-nes.patch.1.pom`
- `bjca-footstone-bpring-data-commons-3.5.13-nes.patch.1.jar`
- `bjca-footstone-bpring-data-commons-3.5.13-nes.patch.1-sources.jar`

The structured installed-POM scan reported `filesScanned=1`, `clean=true`, and
`findings=0`; no internal `cn.bjca.footstone` SNAPSHOT dependency, parent, BOM,
or plugin version remains.

The coordinator then ran an offline local Maven consumer `dependency:tree`
check. It completed successfully in `1.535s` and resolved:

- Data Commons `3.5.13-nes.patch.1`;
- Framework core, beans, and JCL `6.2.19-nes.patch.1`;
- zero internal SNAPSHOT components.

The local publication, full installed-POM scan, and representative consumer
gates are therefore complete. Worktree and release-only diff checks remain
clean; the coordinating session owns the serialized central manifest transition
and dedicated release commit.

## Release commit and Nexus publication

The dedicated release commit is
`837175de73a34f0017b779ad3ae551b6906460ca`. Immediately before deployment,
the complete target publication set was checked again and its POM and JAR were
absent from Nexus RELEASE.

The coordinator executed the incremental Maven deployment once from the
release commit, without `clean` and without compiling or running tests. It
completed successfully in `57.545s`.

Post-deployment verification downloaded the POM and main JAR from Nexus
RELEASE. The remote POM declares Data Commons `3.5.13-nes.patch.1`, Framework
`6.2.19-nes.patch.1`, and zero internal SNAPSHOT references.

- POM URL: `http://192.168.131.36:8088/repository/releases/cn/bjca/footstone/bpring/data/bjca-footstone-bpring-data-commons/3.5.13-nes.patch.1/bjca-footstone-bpring-data-commons-3.5.13-nes.patch.1.pom`
- POM SHA-256: `5aa427e406eafeb8308c202b192900bef4b47b596f5fa900a01936832b8adf71`
- JAR URL: `http://192.168.131.36:8088/repository/releases/cn/bjca/footstone/bpring/data/bjca-footstone-bpring-data-commons/3.5.13-nes.patch.1/bjca-footstone-bpring-data-commons-3.5.13-nes.patch.1.jar`
- JAR SHA-256: `eea3f813c87f951ff142a8a26af7fbb8abc6512f4772df7442fff444ab632749`

An isolated Maven consumer with snapshots disabled resolved the RELEASE-only
graph successfully in `2.070s`. It selected Data Commons
`3.5.13-nes.patch.1` and Framework `6.2.19-nes.patch.1`, with no internal
SNAPSHOT.

Annotated tag `v3.5.13-nes.patch.1` was created locally only after Nexus
verification. Tag object `a7c1db159c48fecf00cb1bf8ae43ef3dfb0e137d`
peels exactly to the release commit. GitHub commit/tag push and remote
verification remain pending because the known `github.com:443` connectivity
blocker persists; Nexus must not be redeployed when the Git operation is
retried.
