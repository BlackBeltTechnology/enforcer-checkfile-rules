# require-glob-matches Specification

## Purpose
The `RequireGlobMatches` enforcer rule verifies that each specified glob pattern matches at least one file under a given root location, using Java NIO's `PathMatcher` and `Files.walkFileTree`.

## Architecture
`RequireGlobMatches` extends `AbstractStandardEnforcerRule` and exposes three configurable fields: `location` (File), `globs` (String[]), and `allowNulls` (boolean). The `execute()` method validates inputs and iterates over globs, delegating to `checkGlobMatchInPath(String, File)` which walks the file tree, relativizes paths against `location`, and matches them with `PathMatcher`. Results are aggregated and thrown as a single `EnforcerRuleException` if any glob has no match.

## Requirements

### Requirement: Globs parameter SHALL be mandatory and non-empty
The rule SHALL reject null or empty `globs` arrays.

#### Scenario: Missing globs parameter
- **GIVEN** a `RequireGlobMatches` rule configuration with no `globs` element
- **WHEN** the enforcer plugin executes the rule
- **THEN** the build SHALL fail with a `NullPointerException` indicating "glob is mandatory"

#### Scenario: Empty globs list
- **GIVEN** a `RequireGlobMatches` rule configuration with an empty `globs` element
- **WHEN** the enforcer plugin executes the rule
- **THEN** the build SHALL fail with an `EnforcerRuleException` indicating "at least 1 glob must be specified"

### Requirement: Rule SHALL pass when all globs match at least one file
The rule SHALL succeed when every listed glob pattern matches at least one file under `location`.

#### Scenario: Glob matches existing files
- **GIVEN** a `location` directory containing `src/foo/bar` and `src/foo/baz`
- **WHEN** the rule is configured with glob `src/foo/b*`
- **THEN** the build SHALL succeed

### Requirement: Rule SHALL fail when a glob matches no files
The rule SHALL fail with a descriptive error when a glob pattern does not match any file under the location.

#### Scenario: Glob does not match
- **GIVEN** a `location` directory containing `src/foo/bar` and `src/foo/baz`
- **WHEN** the rule is configured with glob `src/foo/x*`
- **THEN** the build SHALL fail with an error "Could not find file matches with: src/foo/x* on location:<path>"

### Requirement: Paths SHALL be relativized against the location
The glob matching SHALL operate on paths relative to the configured `location` directory, not absolute paths.

#### Scenario: Relative path matching
- **GIVEN** a `location` set to `${project.basedir}` and a file at `${project.basedir}/src/foo/bar`
- **WHEN** the rule is configured with glob `src/foo/bar`
- **THEN** the glob SHALL match because the path is relativized to `src/foo/bar`

### Requirement: IO errors SHALL be reported gracefully
The rule SHALL catch `IOException` during file tree walking and report a descriptive error rather than propagating the exception.

#### Scenario: IO error during walk
- **WHEN** an `IOException` occurs while walking the file tree for a glob
- **THEN** the rule SHALL report "IO error not find file with: <glob> on location:<path>"

### Requirement: Rule SHALL use a constant cache ID
The rule SHALL return `"0"` from `getCacheId()`, indicating it does not support result caching.

#### Scenario: Cache ID value
- **WHEN** `getCacheId()` is called on a `RequireGlobMatches` instance
- **THEN** the return value SHALL be `"0"`
