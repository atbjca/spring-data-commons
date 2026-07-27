# Local RELEASE verification evidence

Recorded at: `2026-07-27T11:09:37Z`

## Serialized incremental install

The coordinating main session ran this command alone in the global build slot from the component repository:

```bash
JAVA_HOME=/Users/anan/.sdkman/candidates/java/17.0.17-amzn \
JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8 \
MAVEN_OPTS='-Xmx4g -Dfile.encoding=UTF-8' \
./mvnw -DskipTests install
```

- Exit status: success (`0`)
- Maven Total time: `40.632 s`
- Mode: incremental install without `clean`
- Tests: execution skipped by `-DskipTests`; Maven still compiled the test sources
- Concurrency: no other Maven, Gradle, Make, consumer, or deploy command ran concurrently

This command satisfies the mandatory local-install gate without repeating the separately excepted `make build` or `make test` gates.

## Confirmed publication set

The install produced one GAV:

`cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:2.7.18-nes.patch.1`

The local version directory contains these three versioned publication assets:

- `bjca-footstone-bpring-data-commons-2.7.18-nes.patch.1.pom` (`11651` bytes)
- `bjca-footstone-bpring-data-commons-2.7.18-nes.patch.1.jar` (`1360435` bytes)
- `bjca-footstone-bpring-data-commons-2.7.18-nes.patch.1-sources.jar` (`873026` bytes)

Maven's `_remote.repositories` file is local repository bookkeeping and is not a publication asset. There are no additional component GAVs or exclusions.

## Generated POM scan

- `filesScanned`: `1`
- `clean`: `true`
- `findings`: `0`

The generated POM declares the component RELEASE `2.7.18-nes.patch.1`, imports the Framework BOM RELEASE `5.3.39-nes.patch.1`, and contains no internal `cn.bjca.footstone` version ending in `-SNAPSHOT`.

## Offline local consumer

Consumer project:

`/private/tmp/nes-data-commons27-local-consumer/pom.xml`

Exact command:

```bash
mvn -o -f /private/tmp/nes-data-commons27-local-consumer/pom.xml dependency:tree -Dverbose
```

- Exit status: success (`0`)
- Maven Total time: `1.687 s`
- Repository mode: offline local resolution
- Resolved Data Commons: `2.7.18-nes.patch.1` (2.7.18 baseline)
- Resolved Framework core, beans, and JCL: `5.3.39-nes.patch.1` (5.3.39 baseline)
- Internal SNAPSHOT dependencies: `0`

The consumer POM declares only the Data Commons RELEASE, so the Framework results demonstrate transitive resolution through the generated RELEASE metadata.

## Post-gate worktree check

- Branch remains `2.7.x-bjca-patch` at baseline HEAD `44eebbfeec112dcb47719be1a295eb225a8d657e` before the dedicated release commit.
- Production and test source diff remains empty.
- Tracked modifications remain limited to release metadata and component documentation.
- `.claude/` and generated `target/` content remain excluded from release staging.
- Central manifest state was still `prepared` when this component evidence was written; the coordinating main session owns the transition to `locally-verified` and task 3.7.
