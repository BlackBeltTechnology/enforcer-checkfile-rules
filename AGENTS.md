# Enforcer Checkfile Rules - Project Documentation

## Project Overview

**Repository:** BlackBeltTechnology/enforcer-checkfile-rules
**License:** Apache License 2.0
**Java Version:** 21
**Build System:** Maven 3.9.4 with Maven Wrapper (`./mvnw`)

1. Provides custom rules for the Maven Enforcer Plugin that validate file contents and file existence via glob patterns.
2. Ships two rules: `RequireFilesContent` (checks files contain a specific string) and `RequireGlobMatches` (checks glob patterns match files).
3. Rules are auto-discovered by Maven through the `org.apache.maven.plugins.enforcer` package convention — no explicit `implementation` attribute needed in user configuration.
4. Tested exclusively through Maven Invoker integration tests (no unit tests).

## Directory Structure

```
enforcer-checkfile-rules/
├── pom.xml                          # Single-module Maven build (CI-friendly ${revision} versioning)
├── mvnw / mvnw.cmd                  # Maven Wrapper scripts
├── src/
│   ├── main/java/org/apache/maven/plugins/enforcer/
│   │   ├── RequireFilesContent.java # Rule: file content checking
│   │   ├── RequireGlobMatches.java  # Rule: glob pattern matching
│   │   └── package-info.java        # Package docs (explains naming convention)
│   └── it/                          # Integration test scenarios (maven-invoker-plugin)
│       ├── settings.xml             # Shared invoker settings
│       ├── RequireFilesContent/     # 4 test scenarios
│       └── RequireGlobMatches/      # 2 test scenarios
├── .github/                         # Issue templates, CI flow docs
├── .vscode/                         # VS Code settings
├── .zed/                            # Zed editor settings
└── logback-test.xml                 # Test logging config
```

## Core Modules

This is a single-module project (no Maven submodules).

| Component | Type | Purpose |
|-----------|------|---------|
| `RequireFilesContent` | Enforcer Rule | Reads files line-by-line and checks each line with `String.contains(content)`. Supports `allowNulls` to skip null file entries. |
| `RequireGlobMatches` | Enforcer Rule | Walks a directory tree using `Files.walkFileTree` and matches relative paths against `java.nio.file.PathMatcher` glob patterns. |
| `Result` (inner class) | Value Object | Shared pattern in both rules — encapsulates success/failure with error message. Duplicated in each rule class (not extracted). |

## Technology Stack

### Core Technologies
- **Maven Enforcer API** 3.3.0 (`enforcer-api`, `enforcer-rules`) — base classes and plugin integration
- **Maven Core** 3.8.3 (`maven-core`, `maven-artifact`, `maven-plugin-api`) — Maven project model access
- **Guava** 14.0.1 — utility library
- **commons-lang** 2.3 — string utilities
- **Java NIO** (`PathMatcher`, `Files.walkFileTree`) — glob matching in `RequireGlobMatches`

### Build & Quality
- **Maven 3.9.4** with Maven Wrapper
- **JaCoCo** 0.8.12 — code coverage
- **maven-invoker-plugin** 3.6.0 — integration testing
- **flatten-maven-plugin** 1.3.0 — CI-friendly `${revision}` versioning
- **SLF4J** 2.0.16 + **Logback** 1.5.12 — logging (provided scope)

## Build Commands

```bash
# Full build with integration tests
./mvnw clean install

# Compile only
./mvnw compile

# Run integration tests only
./mvnw invoker:install invoker:run

# Skip tests
./mvnw clean install -DskipTests

# Set custom version
./mvnw clean install -Drevision=1.2.3
```

### Maven Profiles

| Profile | Purpose |
|---------|---------|
| `sign-artifacts` | Signs artifacts using `sign-maven-plugin` for release publishing |
| `release-dummy` | Deploys to local `/tmp/` directory for testing release flow |
| `release-judong` | Deploys to Judong Nexus snapshot repository |
| `release-central` | Deploys to Maven Central via Sonatype OSSRH with auto-release |
| `generate-github-asciidoc-diagrams` | Generates PNG diagrams from AsciiDoc using PlantUML |
| `update-source-code-license` | Updates EPL v2 license headers on source files |

## Key Configuration Files

| File | Purpose |
|------|---------|
| `pom.xml` | Build configuration; uses `${revision}` for CI-friendly versioning |
| `logback-test.xml` | Logging configuration for test execution |
| `src/it/settings.xml` | Maven settings used by invoker integration tests |
| `src/it/*/invoker.properties` | Per-scenario config declaring expected build result (`invoker.buildResult`) |

## Development Environment

**Required:**
- Java 21 JDK
- Maven 3.9.4+ (or use included `./mvnw`)

**Package naming convention:** Rules must be in the `org.apache.maven.plugins.enforcer` package so Maven Enforcer discovers them by simple class name without requiring an `implementation` attribute in user configuration.

## Git Workflow

- **Main Branch:** `develop`
- **Versioning:** CI-friendly `${revision}` property (default `1.0.0-SNAPSHOT`), resolved by `flatten-maven-plugin`
- **Branch naming:** `feature/JNG-NUMBER_short_summary` for features, `bugfix/` for fixes
- **Rule:** Every commit must reference a JIRA ticket (e.g., `JNG-xxx`)

## Important Notes

1. There are no unit tests — all testing is done through `maven-invoker-plugin` integration tests in `src/it/`.
2. Both rule classes contain a duplicated inner `Result` class. This is intentional (keeps each rule self-contained).
3. The `org.apache.maven.plugins.enforcer` package name is required for Maven Enforcer auto-discovery. Do not change it.
4. `RequireFilesContent` uses classic `BufferedReader`/`FileReader` (not NIO), while `RequireGlobMatches` uses `java.nio.file` APIs.
5. Always use `./mvnw` instead of a system-installed `mvn` to ensure consistent Maven version.

## Related Documentation

- [README.md](README.md) — Usage examples and rule reference
- [CONTRIBUTING.md](CONTRIBUTING.md) — Contribution guidelines
- [.github/CIFLOW.md](.github/CIFLOW.md) — CI/CD pipeline and branching strategy
