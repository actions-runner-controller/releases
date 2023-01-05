# releases

This repository contains the workflows needed to release new runner images whenever a new version of the runner binaries are released. The images will be pushed to DockerHub and GHCR.

## Usage

### Release new runner images

### Using the CLI

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
| sha | The commit sha to be used to build the runner images. This will be provided to `actions/checkout` & used to tag the container images | '' |
| push_to_registries | Whether to push the images to the registries. Use false to test the build | false |
| troubleshoot | Whether to enable troubleshooting mode. This will start a tmate reverse ssh session | false |

#### Using the UI

You can also trigger the workflow from the UI by clicking on the "Run workflow" button on the [workflow page](https://github.com/actions-runner-controller/releases/actions/workflows/release-runners.yaml).


### Publish controller images

This workflow is triggered whenever a new release is published in [actions/actions-runner-controller](https://github.com/actions/actions-runner-controller). It will build the actions-runner-controller images and push them to DockerHub and GHCR.

```bash
jq -n '{"event_type": "arc", "client_payload": {"release_tag_name": "v0.26.0", "push_to_registries": false}}' \
    | gh api -X POST /repos/actions-runner-controller/releases/dispatches --input -
```

**NOTE:** this workflow should never be triggered manually unless `push_to_registries` is set to false.

### Publish controller canary images

This workflow is triggered whenever a new commit is pushed to the `master` branch in [actions/actions-runner-controller](https://github.com/actions/actions-runner-controller). It will build the actions-runner-controller images and push them to DockerHub and GHCR.

For troubleshooting purposes you can run this workflow with:

```bash
jq -n '{"event_type": "canary", "client_payload": {"sha": "84104de74b8e9e555f530d40d8f33cc9471716f5", "push_to_registries": false}}' \
    | gh api -X POST /repos/actions-runner-controller/releases/dispatches --input -
```

<!-- Table of Paramters -->
| Parameter | Description | Default |
| --- | --- | --- |
| sha | The commit sha to be used to build the runner images. This will be provided to `actions/checkout` & used to tag the container images  | Required. Cannot be empty |
| push_to_registries | Whether to push the images to the registries. Use false to test the build | Required. Cannot be empty |

**NOTE:** this workflow should never be triggered manually unless `push_to_registries` is set to false.

### Publish helm charts

This workflow is triggered by [this workflow](https://github.com/actions/actions-runner-controller/blob/master/.github/workflows/publish-chart.yaml) whenever a new helm chart needs to be published.

```bash
jq -n '{"event_type": "chart"}' \
    | gh api -X POST /repos/actions-runner-controller/releases/dispatches --input -
```