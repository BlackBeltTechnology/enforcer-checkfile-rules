# CI/CD Flow — Development Version and Branch Handling

This document describes the branching strategy, versioning policy, and GitHub Actions CI/CD pipeline used by this project.

## Branches

The branching model is based on [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow):

| Branch Pattern | Purpose |
|---------------|---------|
| `develop` | Main development branch; contains latest development sources |
| `feature/JNG-NUMBER_short_summary` | Feature branches based on `develop`; merged back when complete |
| `release/X.Y.Z` or `X_Y_Z` | Release branches cut from `develop` for stabilization |
| `bugfix/JNG-NUMBER_short_summary` | Bug fixes applied to release branches, then merged forward |
| `support/JNG-NUMBER_short_summary` | Minor changes to previous releases; merged back to release branch |
| `hotfix/JNG-NUMBER_short_summary` | Critical fixes applied to both `master` and release branches |
| `master` | Latest released sources |

### Branch Flow

```mermaid
gitGraph
    commit id: "init"
    branch develop
    commit id: "dev-1"
    branch feature/JNG-1
    commit id: "feat-1"
    commit id: "feat-2"
    checkout develop
    merge feature/JNG-1 id: "merge-feat"
    branch release/1.0-beta1
    commit id: "rc-1"
    branch bugfix/JNG-4
    commit id: "fix-1"
    checkout release/1.0-beta1
    merge bugfix/JNG-4 id: "merge-fix"
    checkout develop
    merge release/1.0-beta1 id: "merge-release"
    checkout master
    merge release/1.0-beta1 id: "release-1.0"
```

## Version Numbers

Versions follow semantic versioning with these rules:

| Event | Version Change |
|-------|---------------|
| Start a `feature/` branch | No change (inherits from `develop`) |
| Cut a `release/` branch from `develop` | 2nd number incremented on `develop` |
| Start a `bugfix/` branch | No change (applied to release branch before merge to `master`) |
| Start a `support/` branch | 3rd number incremented |
| Start a `hotfix/` branch | 4th number incremented |

## GitHub Actions Workflows

The CI/CD pipeline consists of four interconnected workflows:

### Overview

```mermaid
flowchart TD
    subgraph Triggers
        Push[Push to develop]
        PR[PR to develop / master / release]
        Manual[Manual trigger with version]
        MergeTag[Push merge-pr/* tag]
        MasterPush[Push to master]
    end

    subgraph Workflows
        Build[build.yml]
        MergePR[merge-pr-tagged.yml]
        CreateRelease[create-release-on-master.yml]
        Release[release.yml]
    end

    Push --> Build
    PR --> Build
    Build -->|release branch PR| MergeTag
    MergeTag --> MergePR
    MergePR -->|major.minor.qualifier| MasterPush
    MasterPush --> CreateRelease
    MergePR -->|other format| Build
    Manual --> Release
    Release -->|creates PRs| Build
```

### build.yml

Triggered on pushes to `develop` or pull requests to `develop`, `master`, `increment/*`, or `release/*` branches.

```mermaid
flowchart TD
    Start([Push / PR]) --> BranchCheck{Base branch?}
    BranchCheck -->|master, release/*| SetRelVersion[Set version from pom.xml<br/>without -SNAPSHOT]
    BranchCheck -->|develop, increment/*| SetDevVersion[Set version as<br/>major.minor.qualifier.date_commitId_branch]
    SetRelVersion --> BuildDeploy[Build and deploy to Nexus]
    SetDevVersion --> BuildDeploy
    BuildDeploy --> CreateTag[Create git tag v-version]
    CreateTag --> IsPR{increment/* or release/*?}
    IsPR -->|Yes| MergeTag[Create merge-pr/version tag<br/>triggers merge-pr-tagged.yml]
    IsPR -->|No| IsDevelop{develop?}
    IsDevelop -->|Yes| Changelog[Build changelog<br/>Create GitHub prerelease]
    IsDevelop -->|No| Done([End])
    MergeTag --> Done
    Changelog --> Done
```

### merge-pr-tagged.yml

Triggered when a `merge-pr/*` tag is pushed.

```mermaid
flowchart TD
    Start([merge-pr/* tag pushed]) --> GetVersion[Extract version from tag]
    GetVersion --> VersionCheck{Version format?}
    VersionCheck -->|major.minor.qualifier| MergeMaster[Merge PR to master<br/>triggers create-release-on-master.yml]
    VersionCheck -->|other| SquashDevelop[Squash PR to develop<br/>triggers build.yml]
    MergeMaster --> Cleanup[Delete merge-pr/* tag]
    SquashDevelop --> Cleanup
```

### create-release-on-master.yml

Triggered on pushes to `master`.

```mermaid
flowchart TD
    Start([Push to master]) --> GetVersion[Get version from tag]
    GetVersion --> Changelog[Build changelog]
    Changelog --> Release[Create GitHub release<br/>marked as latest]
```

### release.yml

Triggered manually with a version parameter (`auto` or `major.minor.qualifier`).

```mermaid
flowchart TD
    Start([Manual trigger]) --> VersionCheck{Given version?}
    VersionCheck -->|auto| FromPom[Set release version<br/>from pom.xml without -SNAPSHOT]
    VersionCheck -->|specific| UseGiven[Set release version<br/>to given value]
    FromPom --> CalcNext[Set next version =<br/>release qualifier + 1]
    UseGiven --> CalcNext
    CalcNext --> PRMaster[Create PR to master<br/>with release version]
    CalcNext --> PRDevelop[Create PR to develop<br/>with next version]
    PRMaster -->|triggers| Build[build.yml]
    PRDevelop -->|triggers| Build
```

## Development Rules

> **Important:** There is no commit without a ticket number. Every pull request and commit must reference a JIRA ticket (e.g., `JNG-xxx`).

Issue tracking: [JIRA Dashboard](https://blackbelt.atlassian.net/jira/dashboards)
