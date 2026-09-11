# `action-deploy-docker` - **Github Action**

Builds a Docker image for the calling repository and publishes it to a container
registry, tagged by commit SHA.

Two entry points are provided:

| Entry point | Shape | Multi-platform builds |
| ----------- | ----- | --------------------- |
| [Reusable workflow](#reusable-workflow-recommended) | Several jobs | Each platform on its own runner, in parallel, natively |
| [Composite action](#composite-action) | Single step | All platforms in one job, non-native ones under QEMU |

Both default to publishing a `linux/amd64,linux/arm64` image on `push` events
and to just building `linux/amd64` on pull requests.

Images are tagged `sha-<short sha>`, or `sha-<short sha>-<branch>` on `epic/*`
branches.

## Reusable workflow (recommended)

Runs one `build` job per platform: platforms listed in the `runners` map build
natively on that runner, any other Linux platform is built under QEMU on
`ubuntu-latest`. Every platform image is pushed by digest and a final `merge`
job assembles the manifest list under the tags above.

The calling job must grant the permissions the workflow needs; the default
`GITHUB_TOKEN` cannot push to GHCR unless `packages: write` is granted.

```yaml
name: Build and publish Docker image

on:
  push:
    branches: ['master', 'epic/**']
  pull_request:
    branches: ['**']

jobs:
  build-image:
    uses: valitydev/action-deploy-docker/.github/workflows/build-image.yml@v3
    permissions:
      contents: read
      packages: write
    with:
      registry-username: ${{ github.actor }}
    secrets:
      registry-access-token: ${{ secrets.GITHUB_TOKEN }}
```

Note the input types: `push` and `platforms` are strings (`push: 'false'`),
`use-env-buildargs` is a boolean (`use-env-buildargs: false`). If arm runners
are not available to the repository, pass
`runners: '{"linux/amd64":"ubuntu-latest"}'` to build arm64 under QEMU instead.

### Inputs

| Name                | Default                                                             | Description                                                                                            |
| ------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `registry-username` | _required_                                                          | Username for image registry                                                                            |
| `docker-registry`   | `ghcr.io`                                                           | Docker image registry                                                                                  |
| `context-path`      | `.`                                                                 | Build's context path                                                                                   |
| `dockerfile-path`   | `./Dockerfile`                                                      | Path to Dockerfile                                                                                     |
| `push`              | `true` on `push` events, `false` otherwise                          | Push built image to the registry? (string: `'true'` / `'false'`)                                       |
| `platforms`         | `linux/amd64,linux/arm64` on `push` events, `linux/amd64` otherwise | Comma-separated list of target platforms                                                               |
| `use-env-buildargs` | `true`                                                              | Source docker build args from `.env` in the repository root? (boolean; literal `KEY=value` lines)      |
| `runners`           | `{"linux/amd64":"ubuntu-latest","linux/arm64":"ubuntu-24.04-arm"}`  | JSON map of platform to the runner label that builds it natively; unmapped platforms fall back to QEMU |

### Secrets

| Name                    | Description                     |
| ----------------------- | ------------------------------- |
| `registry-access-token` | Access token for image registry |

## Composite action

Single-job variant. Use it when the image must be built in the same job as
other steps, e.g. right after compiling an artifact that the Dockerfile copies.
Non-native platforms are emulated with QEMU, which is slow for images that
compile code inside the Dockerfile.

```yaml
jobs:
  build-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: valitydev/action-deploy-docker@v3
        with:
          registry-username: ${{ github.actor }}
          registry-access-token: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Name                    | Default                                                             | Description                                |
| ----------------------- | ------------------------------------------------------------------- | ------------------------------------------ |
| `registry-username`     | _required_                                                          | Username for image registry                |
| `registry-access-token` | _required_                                                          | Access token for image registry            |
| `docker-registry`       | `ghcr.io`                                                           | Docker image registry                      |
| `context-path`          | `.`                                                                 | Build's context path                       |
| `dockerfile-path`       | `./Dockerfile`                                                      | Path to Dockerfile                         |
| `push`                  | `true` on `push` events, `false` otherwise                          | Push built image to the registry?          |
| `platforms`             | `linux/amd64,linux/arm64` on `push` events, `linux/amd64` otherwise | Comma-separated list of target platforms   |
| `use-env-buildargs`     | `true`                                                              | Source docker build args from `.env` file? |
