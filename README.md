# ESP-IDF projects CI workflow

Shared composite actions and reusable workflows for ESP-IDF component projects.

## Actions

- [Submodule cache](actions/submodule-cache/README.md) initializes git
  submodules shallow, in parallel and with retries, using a cache shared by
  all jobs in a workflow run.
- [Setup ESP-IDF build environment](actions/setup-idf-build-env/README.md)
  installs the shared CI Python tools and configures compiler warning flags.

## Reusable workflows

The ESP-IDF component project CI pipeline is available as two
[reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows):

- `.github/workflows/build_and_run_apps.yml` — the full pipeline: change
  detection from PR labels and diffs, the submodule cache populate job, one
  `reusable-ci.yml` invocation per SDK version, aggregate reports and a
  stable `CI Gate` status check.
- `.github/workflows/reusable-ci.yml` — the per-SDK pipeline: coverage
  audit, per-target build shards, test planning, hardware/QEMU/host test
  workers, JUnit and HTML reports. Can also be called directly.

## Consumer setup

Workflow triggers must live in the consuming repository, so each consumer
keeps a thin caller workflow:

```yaml
name: Build and Run Apps

on:
  pull_request:
    types: [opened, reopened, synchronize]
  push:
    branches:
      - master

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  pipeline:
    name: CI pipeline
    permissions:
      contents: read
      pull-requests: write
      checks: write
      actions: read
    uses: espressif/esp-idf-projects-ci-workflow/.github/workflows/build_and_run_apps.yml@main
    with:
      idf_versions: '["release-v5.4", "release-v6.0", "latest"]'
```

Consumers supply in their own repository:

- the `idf_versions` input — JSON array of `espressif/idf` Docker tags to
  build and test against
- `.idf_build_apps.toml`, `.build-test-rules.yml`, manifests
- pytest configuration (`pytest.ini`, root `conftest.py`) and test apps

The pipeline checks out the consumer repository and runs in its context:
caches, artifacts, PR labels (`PR: test all apps`, `PR: build only`) and
tokens are the consumer's. CI helpers (`.github/actions/ci-tools`,
`actions/setup-idf-build-env`, `.github/get_ci_changes.py`) are fetched from
the exact revision of the called workflow, so every consumer runs the same
tooling. The caller workflow above documents the permission superset the
called jobs need; job-level permissions inside the pipeline narrow it further.

Called jobs report with the caller job's name as prefix, so the stable
status check for branch protection is `CI pipeline / CI Gate` — keep the
caller job name unchanged, and never require matrix job names directly
(they change with the `idf_versions` input).

Pipeline behavior (change selection, shards, planning, reports, artifacts)
is documented in
[idf-extra-components](https://github.com/espressif/idf-extra-components/blob/master/.github/readme_workflows.md).
