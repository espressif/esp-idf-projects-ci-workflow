# Setup ESP-IDF build environment

Composite action used by the ESP-IDF CI workflows to install the shared Python
tools and export the compiler warning flags used by all CI builds. It keeps the
`idf-component-manager` version supplied by the selected SDK.

## Usage

```yaml
- uses: espressif/esp-idf-projects-ci-workflow/actions/setup-idf-build-env@main
  with:
    target: esp32
```

## Inputs

| Input | Default | Meaning |
| --- | --- | --- |
| `target` | Required | ESP-IDF target; `linux` selects the host compiler for warning checks |

The selected ESP-IDF environment must be available through `IDF_PATH`. The
action sources `IDF_PATH/export.sh` and requires `bash`, Python and `pip`.
