# Contributing to JUDO

## Development Environment

Make sure your development environment meets the requirements described in the parent project's [CONTRIBUTING guide](https://github.com/BlackBeltTechnology/judo-community/blob/develop/CONTRIBUTING.adoc):

- **Java 21** JDK
- **Maven 3.9.4+** (or use the included `./mvnw` wrapper)

## Project Structure

This is a single-module Maven project. The main source code lives in:

```
src/main/java/org/apache/maven/plugins/enforcer/
```

There are no unit tests. Validation is done through integration tests in `src/it/` using `maven-invoker-plugin`.

## Commands

### Run full build (compile + integration tests)

```sh
./mvnw clean install
```

### Run tests only

```sh
./mvnw clean test
```

### Run integration tests only

```sh
./mvnw invoker:install invoker:run
```

## Submitting an Issue

Before submitting, search the [issue tracker](https://github.com/BlackBeltTechnology/enforcer-checkfile-rules/issues) for existing reports.

To help us reproduce and fix bugs quickly, please include:

- Output of `java -version` and `mvn -version`
- Relevant `pom.xml` or `.flattened-pom.xml`
- A minimal use-case that fails

File new issues using the [issue form](https://github.com/BlackBeltTechnology/enforcer-checkfile-rules/issues/new/choose).

## Submitting a PR

This project follows [GitHub's standard forking model](https://guides.github.com/activities/forking/). Fork the project and submit pull requests from your fork.
