
**BREAKING CHANGES in v1.0:**
* **node_version is now required (previously defaulted to 16)**
* **sonar_comment_token has been removed (ignored by SonarCloud)**
* **sonar_project_token has been renamed sonar_token**

<!-- Badges -->
[![Issues](https://img.shields.io/github/issues/bcgov-nr/action-test-and-analyse)](/../../issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/bcgov-nr/action-test-and-analyse)](/../../pulls)
[![MIT License](https://img.shields.io/github/license/bcgov-nr/action-test-and-analyse.svg)](/LICENSE)
[![Lifecycle](https://img.shields.io/badge/Lifecycle-Experimental-339999)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)

<!-- Reference-Style link -->
[SonarCloud]: https://sonarcloud.io
[Issues]: https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue
[Pull Requests]: https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/working-with-your-remote-repository-on-github-or-github-enterprise/creating-an-issue-or-pull-request

# Unit Test (nodejs), Coverage and Analysis with SonarCloud

This action runs unit tests and optionally runs analysis, including coverage, using [SonarCloud](https://sonarcloud.io).  SonarCloud can be configured to comment on pull requests or stop failing workflows.

Only nodejs (JavaScript, TypeScript) is currently supported, with plans for Java next.

# Usage

```yaml
- uses: bcgov-nr/action-test-and-analyse@main
  with:
    ### Required

    # Commands to run unit tests
    # Please configure your app to generate coverage (coverage/lcov.info)
    commands: |
      npm ci
      npm run test:cov

    # Project/app directory
    dir: frontend

    # Node.js version
    # BREAKING CHANGE: previously defaulted to 16 (LTS)
    node_version: "20"

    ### Typical / recommended

    # Sonar arguments
    # https://docs.sonarcloud.io/advanced-setup/analysis-parameters/
    sonar_args: |
        -Dsonar.exclusions=**/coverage/**,**/node_modules/**
        -Dsonar.organization=bcgov-sonarcloud
        -Dsonar.projectKey=bcgov_${{ github.repository }}

    # Sonar token
    # Available from sonarcloud.io or your organization administrator
    # BCGov i.e. https://github.com/BCDevOps/devops-requests/issues/new/choose
    # Provide an unpopulated token for pre-setup, section will be skipped
    sonar_token:
      description: ${{ secrets.SONAR_TOKEN }}

    ### Usually a bad idea / not recommended

    # Overrides the default branch to diff against
    # Defaults to the default branch, usually `main`
    diff_branch: ${{ github.event.repository.default_branch }}

    # Repository to clone and process
    # Useful for consuming other repos, like in testing
    # Defaults to the current one
    repository: ${{ github.repository }}

    # Branch to clone and process
    # Useful for consuming non-default branches, like in testing
    # Defants to empty, cloning the default branch
    branch: ""
```

# Example, Single Directory with SonarCloud Analysis

Run unit tests and provide results to SonarCloud.  This is a full workflow that runs on pull requests, merge to main and workflow_dispatch.  Use a GitHub Action secret to provide ${{ secrets.SONAR_TOKEN }}.

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/unit-tests.yml`

Note: Provde an unpopulated SONAR_TOKEN until one is provisioned.  SonarCloud will only run once populated, allowing for pre-setup.

```yaml
name: Unit Tests and Analysis

on:
  pull_request:
  push:
    branches:
      - main
    paths-ignore:
      - ".github/**"
      - "**.md"
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  tests:
    name: Run Unit Tests and Analyse
    runs-on: ubuntu-22.04
    steps:
      - uses: bcgov-nr/action-test-and-analyse@main
        with:
          commands: |
            npm ci
            npm run test:cov
          dir: frontend
          node_version: "20"
          sonar_args: |
            -Dsonar.exclusions=**/coverage/**,**/node_modules/**
            -Dsonar.organization=bcgov-nr
            -Dsonar.projectKey=bcgov-nr_action-test-and-analyse_frontend
          sonar_token: ${{ secrets.SONAR_TOKEN }}
```

# Example, Single Directory, Only Running Unit Tests (No SonarCloud)

Run unit tests, but not SonarCloud.

```yaml
jobs:
  tests:
    name: Run Unit Tests and Analyse
    runs-on: ubuntu-22.04
    steps:
      - uses: bcgov-nr/action-test-and-analyse@main
        with:
          commands: |
            npm ci
            npm run test:cov
          dir: frontend
          node_version: "20"
```

# Example, Matrix / Multiple Directories with Sonar Cloud

Unit test and analyze projects in multiple directories in parallel.  This time `repository` and `branch` are provided.  Please note how secrets must be passed in to composite Actions using the secrets[matrix.variable] syntax.

```yaml
jobs:
  tests:
    name: Unit Tests
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        dir: [backend, frontend]
        include:
          - dir: backend
            token: SONAR_TOKEN_BACKEND
          - dir: frontend
            token: SONAR_TOKEN_FRONTEND
    steps:
      - uses: actions/checkout@v3
      - uses: bcgov-nr/action-test-and-analyse@main
        with:
          commands: |
            npm ci
            npm run test:cov
          dir: ${{ matrix.dir }}
          node_version: "20"
          sonar_args: |
            -Dsonar.exclusions=**/coverage/**,**/node_modules/**
            -Dsonar.organization=bcgov-nr
            -Dsonar.projectKey=bcgov-nr_action-test-and-analyse_${{ matrix.dir }}
          sonar_token: ${{ secrets[matrix.token] }}
          repository: bcgov/nr-quickstart-typescript
          branch: main
```

# Sonar Project Token

SonarCloud project tokens are free, available from [SonarCloud] or your organization's aministrators.

For BC Government projects, please create an [issue for our platform team](https://github.com/BCDevOps/devops-requests/issues/new/choose).

After sign up, a token should be available from your project on the [SonarCloud] site.  Multirepo projects (e.g. backend, frontend) will have multiple projects.  Click `Administration > Analysis Method > GitHub Actions (tutorial)` to find yours.

E.g. https://sonarcloud.io/project/configuration?id={<PROJECT>}&analysisMode=GitHubActions

# Feedback

Please contribute your ideas!  [Issues] and [pull requests] are appreciated.

<!-- # Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia. -->
