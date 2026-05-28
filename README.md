# kannich-action

A GitHub Action for running [Kannich](https://kannich.dev) build executions with automatic caching.

## Prerequisites

- A Linux runner (`ubuntu-latest` or similar). Windows runners are not supported.
- The `kannichw` wrapper script committed to your repository. See the [Kannich documentation](https://kannich.dev/docs/quick-start/#installation) for setup instructions.

## Basic usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # always pin actions to a SHA hash to mitigate against supply chain attacks
      - uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

      - uses: derkork/kannich-action@302322e072715e74738f589ead3c167a5735ae35 #v1
        with:
          image: derkork/kannich@sha256:a23067a63c1943b4f24a96bf9841279c2895ce97cc7565133773f6814ae8281e # 0.11.0-0.10.0
          executions: |
            build
            test
```

Each line of `executions` is a separate call to `./kannichw`. If one execution fails, the following ones are skipped.

## Inputs

| Input           | Required | Default                            | Description                                                                                                                                            |
|-----------------|----------|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `image`         | yes      | -                                  | The Kannich Docker image to use. Accepts tag notation (`derkork/kannich:0.11.0-0.10.0`) and digest notation (`derkork/kannich@sha256:...`).            |
| `executions`    | yes      | -                                  | Executions to run, one per line. Each non-empty line is passed as arguments to `kannichw`.                                                             |
| `env`           | no       | -                                  | Extra environment variables in `KEY=VALUE` format, one per line. Use this for Kannich configuration variables or any variables your kannichfile needs. |
| `kannichw-path` | no       | `${{ github.workspace }}/kannichw` | Path to the `kannichw` script. Override when `kannichw` is not at the repository root.                                                                 |
| `cache-dir`     | no       | `~/.kannich-cache`                 | Host directory used as the Kannich dependency cache.                                                                                                   |

## Pinning the image

Always pin `image` to a specific version rather than using `:latest`, so your builds are repeatable. To mitigate against supply chain attacks, we recommend using a sha256 digest:

```yaml
image: derkork/kannich@sha256:a23067a63c1943b4f24a96bf9841279c2895ce97cc7565133773f6814ae8281e #0.11.0-0.10.0
```

Using a version tag is also acceptable if you prefer readable version numbers:

```yaml
image: derkork/kannich:0.11.0-0.10.0
```

## Caching

The action automatically saves and restores the Kannich build cache between runs using `actions/cache`. The cache is keyed by operating system and accumulates over time - there is no need to hash dependency files because Kannich will re-fetch anything that is missing. The cache is saved even when the build fails.

## Passing environment variables

Use the `env` input to pass any additional environment variables to Kannich:

```yaml
- uses: derkork/kannich-action@302322e072715e74738f589ead3c167a5735ae35 # v1
  with:
    image: derkork/kannich@sha256:<digest>
    executions: build
    env: |
      MY_CUSTOM_VAR=some-value
      MY_OTHER_VAR=some-other-value
```

Lines starting with `#` and empty lines are ignored.

## Changing the `kannichw` path

If `kannichw` is not at the repository root, set `kannichw-path`:

```yaml
- uses: derkork/kannich-action@302322e072715e74738f589ead3c167a5735ae35 # v1
  with:
    image: derkork/kannich@sha256:a23067a63c1943b4f24a96bf9841279c2895ce97cc7565133773f6814ae8281e #0.11.0-0.10.0
    kannichw-path: ./services/backend/kannichw
    executions: build
```
