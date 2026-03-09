# require-files-content Specification

## Purpose
The `RequireFilesContent` enforcer rule verifies that a list of specified files exist and each contains a given string, checked line by line using `String.contains()`.

## Architecture
`RequireFilesContent` extends `AbstractStandardEnforcerRule` and exposes three configurable fields: `content` (String), `files` (File[]), and `allowNulls` (boolean). The `execute()` method validates inputs, iterates over files, and delegates to `checkFile(File)` which returns a `Result` value object. Errors are aggregated and thrown as a single `EnforcerRuleException`.

## Requirements

### Requirement: Content parameter SHALL be mandatory
The rule SHALL throw a `NullPointerException` when `content` is not specified.

#### Scenario: Missing content parameter
- **GIVEN** a `RequireFilesContent` rule configuration with no `content` element
- **WHEN** the enforcer plugin executes the rule
- **THEN** the build SHALL fail with a `NullPointerException` indicating "content is mandatory"

### Requirement: Files parameter SHALL be mandatory and non-empty
The rule SHALL reject null or empty `files` arrays.

#### Scenario: Missing files parameter
- **GIVEN** a `RequireFilesContent` rule configuration with no `files` element
- **WHEN** the enforcer plugin executes the rule
- **THEN** the build SHALL fail with a `NullPointerException` indicating "file is mandatory"

#### Scenario: Empty files list
- **GIVEN** a `RequireFilesContent` rule configuration with an empty `files` element
- **WHEN** the enforcer plugin executes the rule
- **THEN** the build SHALL fail with an `EnforcerRuleException` indicating "at least 1 file must be specified"

### Requirement: Rule SHALL pass when all files contain the specified content
The rule SHALL succeed when every listed file contains at least one line matching the `content` string via `String.contains()`.

#### Scenario: Content found in all files
- **GIVEN** files `src/foo/bar` and `src/foo/baz` each containing a line with "AutoUpdate:true"
- **WHEN** the rule is configured with `content` = "AutoUpdate:true" and both files listed
- **THEN** the build SHALL succeed

### Requirement: Rule SHALL fail when any file does not contain the content
The rule SHALL fail with a descriptive error when at least one file does not contain the specified string.

#### Scenario: Content not found in a file
- **GIVEN** files `src/foo/bar` and `src/foo/baz` where `baz` does not contain "AutoUpdate:true"
- **WHEN** the rule is configured with `content` = "AutoUpdate:true" and both files listed
- **THEN** the build SHALL fail with an error message containing the path of `baz` and `Doesn't contain: "AutoUpdate:true"`

### Requirement: Rule SHALL fail when a file does not exist
The rule SHALL report "Not a file" for entries that point to non-existent paths.

#### Scenario: File does not exist
- **GIVEN** a `files` entry pointing to a path that does not exist on disk
- **WHEN** the enforcer plugin executes the rule
- **THEN** the build SHALL fail with an error containing "Not a file"

### Requirement: Null file entries SHALL be handled according to allowNulls
When `allowNulls` is `true`, null file entries SHALL be treated as passing. When `false` (default), null entries SHALL cause failure.

#### Scenario: Null file with allowNulls=true
- **GIVEN** a `files` list containing a null entry and `allowNulls` set to `true`
- **WHEN** the enforcer plugin executes the rule
- **THEN** the null entry SHALL be treated as a success

#### Scenario: Null file with allowNulls=false
- **GIVEN** a `files` list containing a null entry and `allowNulls` set to `false` (default)
- **WHEN** the enforcer plugin executes the rule
- **THEN** the rule SHALL report "Empty file name was given and allowNulls is set to false"
