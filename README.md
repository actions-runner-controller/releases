# releases

This repository contains the workflows needed to release new runner images whenever a new version of the runner binaries are released. The images will be pushed to DockerHub and GHCR.

The main reason why these workflows were extracted is that pushing container images to GHRC with GitHub App authentication is not possible at the moment.

## Usage

### Release new runner images (release-runners.yaml)

```mermaid
flowchart LR
    workflow["release-runners.yaml"] -- workflow_dispatch* --> workflow_b["release-runners.yaml"]
    subgraph repository: actions/actions-runner-controller
    runner_updates_check["arc-update-runners-scheduled.yaml"] -- "polls (daily)" --> runner_releases["actions/runner/releases"]
    runner_updates_check -- creates --> runner_update_pr["PR: update /runner/VERSION"]
    runner_update_pr --> runner_update_pr_merge{{"merge"}}
    runner_update_pr_merge -- triggers --> workflow["release-runners.yaml"]
    end
    subgraph repository: actions-runner-controller/releases
    workflow_b["release-runners.yaml"] -- push --> A["GHCR: \n actions-runner-controller/actions-runner:* \n actions-runner-controller/actions-runner-dind:* \n actions-runner-controller/actions-runner-dind-rootless:*"]
    workflow_b["release-runners.yaml"] -- push --> B["DockerHub: \n summerwind/actions-runner:* \n summerwind/actions-runner-dind:* \n summerwind/actions-runner-dind-rootless:*"]
    event_b{{"workflow_dispatch"}} -- triggers --> workflow_b["release-runners.yaml"]
    end
```

You can trigger the workflow from the CLI using the following command:

```bash
gh workflow run release-runners.yaml -R actions-runner-controller/releases \
    -f runner_version=2.300.2 \
    -f docker_version=20.10.12 \
    -f runner_container_hooks_version=0.2.0 \
    -f sha='3acef9e2863a2585142a566e78e5840b9cd22d9a' \
    -f push_to_registries=true
```

<!-- Table of Paramters -->
| Parameter                        | Description                                                                                                                                                                                                                         | Default       |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| `runner_version`                 | The version of the [actions/runner](https://github.com/actions/runner) to use                                                                                                                                                       | `2.300.2`     |
| `docker_version`                 | The version of docker to use                                                                                                                                                                                                        | `20.10.12`    |
| `runner_container_hooks_version` | The version of [actions/runner-container-hooks](https://github.com/actions/runner-container-hooks) to use                                                                                                                           | `0.2.0`       |
| `sha`                            | The commit sha from [actions/actions-runner-controller](https://github.com/actions/actions-runner-controller) to be used to build the runner images. This will be provided to `actions/checkout` & used to tag the container images | Empty string. |
| `push_to_registries`             | Whether to push the images to the registries. Use false to test the build                                                                                                                                                           | false         |

### Publish (arc) controller images (publish-arc.yaml)

```mermaid
flowchart LR
    subgraph repository: actions/actions-runner-controller
    event_a{{"release: published"}} -- triggers --> workflow_a["arc-publish.yaml"]
    event_b{{"workflow_dispatch"}} -- triggers --> workflow_a["arc-publish.yaml"]
    workflow_a["arc-publish.yaml"] -- uploads --> package["actions-runner-controller.tar.gz"]
    end
    subgraph repository: actions-runner-controller/releases
    workflow_a["arc-publish.yaml"] -- triggers --> event_d{{"repository_dispatch"}} --> workflow_b["publish-arc.yaml"]
    workflow_b["publish-arc.yaml"] -- push --> A["GHCR: \nactions-runner-controller/actions-runner-controller:*"]
    workflow_b["publish-arc.yaml"] -- push --> B["DockerHub: \nsummerwind/actions-runner-controller:*"]
    end
```

This workflow is triggered whenever a new release is published in [actions/actions-runner-controller](https://github.com/actions/actions-runner-controller). It will build the actions-runner-controller images and push them to DockerHub and GHCR.

```bash
jq -n '{"event_type": "arc", "client_payload": {"release_tag_name": "v0.26.0", "push_to_registries": false}}' \
    | gh api -X POST /repos/actions-runner-controller/releases/dispatches --input -
```

| Parameter | Description | Default |
| --- | --- | --- |
| release_tag_name | The controller image tag to use. This is not a git tag. | Empty string. Field required. |
| push_to_registries | Whether to push the image to the registries. Use false to test the build | false |

**NOTE:** this workflow should never be triggered manually unless `push_to_registries` is set to false. Otherwise, built images will be pushed to the registries.

### Publish canary images (publish-canary.yaml)

```mermaid
flowchart LR
    subgraph repository: actions/actions-runner-controller
    event_a{{"push: [master]"}} -- triggers --> workflow_a["publish-canary.yaml"]
    end
    subgraph repository: actions-runner-controller/releases
    workflow_a["publish-canary.yaml"] -- triggers --> event_d{{"repository_dispatch"}} --> workflow_b["publish-canary.yaml"]
    workflow_b["publish-canary.yaml"] -- push --> A["GHCR: \nactions-runner-controller/actions-runner-controller:canary"]
    workflow_b["publish-canary.yaml"] -- push --> B["DockerHub: \nsummerwind/actions-runner-controller:canary"]
    end
```

This workflow is triggered whenever a new commit is pushed to the `master` branch in [actions/actions-runner-controller](https://github.com/actions/actions-runner-controller). It will build the actions-runner-controller images and push them to DockerHub and GHCR.

For troubleshooting purposes you can run this workflow with:

```bash
jq -n '{"event_type": "canary", "client_payload": {"sha": "84104de74b8e9e555f530d40d8f33cc9471716f5", "push_to_registries": false}}' \
    | gh api -X POST /repos/actions-runner-controller/releases/dispatches --input -
```

<!-- Table of Paramters -->
| Parameter | Description | Default |
| --- | --- | --- |
| sha | The commit sha to be used to build the runner images. This will be provided to `actions/checkout` & used to tag the container images  | Empty string. Field required. |
| push_to_registries | Whether to push the images to the registries. Use false to test the build | false |

**NOTE:** this workflow should never be triggered manually unless `push_to_registries` is set to false.
