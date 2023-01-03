# releases

This repository contains the workflows needed to release new runner images whenever a new version of the runner binaries are released. The images will be pushed to DockerHub and GHCR.

## Usage

### Triggering a new release using gh cli

You can trigger the workflow from the CLI using the following command:

```bash
gh workflow run release-runners.yaml -R actions-runner-controller/releases \
    -f runner_version=2.300.2 \
    -f docker_version=20.10.12 \
    -f runner_container_hooks_version=0.2.0 \
    -f sha='3acef9e2863a2585142a566e78e5840b9cd22d9a' \
    -f push_to_registries=true \
    -f troubleshoot=true
```

<!-- Table of Paramters -->
| Parameter | Description | Default |
| --- | --- | --- |
| runner_version | The version of the runner binaries to use | `2.300.2` |
| docker_version | The version of the docker binaries to use | `20.10.12` |
| runner_container_hooks_version | The version of the runner container hooks to use | `0.2.0` |
| sha | The commit sha to be used to build the runner images. This will be provided to `actions/checkout`  | '' |
| push_to_registries | Whether to push the images to the registries. Use false to test the build | false |
| troubleshoot | Whether to enable troubleshooting mode. This will start a tmate reverse ssh session | false |

### Using the UI

You can also trigger the workflow from the UI by clicking on the "Run workflow" button on the [workflow page](https://github.com/actions-runner-controller/releases/actions/workflows/release-runners.yaml).
