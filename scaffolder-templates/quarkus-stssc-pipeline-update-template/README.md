# Quarkus Trusted Software Supply Chain Pipeline Update Template

This Backstage template creates a GitLab **merge request** that updates an **existing** application repository to use a Red Hat **Trusted Software Supply Chain** (Trusted Application Pipeline).

Two files are updated:

* `.tekton/on-push.yaml` — updates the `pipelinesascode.tekton.dev/pipeline` annotation to point to the Trusted Software Supply Chain pipeline definition (task references and the rest of the PipelineRun stay unchanged).
* `Dockerfile` — updates the base image used in the `FROM` instruction.

## How it works

1. **Parameters** — the user provides the target repository (owner + name), the branch to target, the **Trusted Software Supply Chain pipeline** to reference (choice list), the new Docker base image, and merge request options.
2. **`catalog:fetch`** — reads `component:default/provisioning-data` to resolve the GitLab host and Quay host of the cluster (same convention as the other tssc templates).
3. **`fetch:template`** — renders `skeleton/.tekton/on-push.yaml` and `skeleton/Dockerfile` with the provided values.
4. **`publish:gitlab:merge-request`** — pushes the two files to a dedicated source branch (created from the target branch) and opens the GitLab merge request.

## Trusted Software Supply Chain pipeline choices

The `pipelineName` parameter is a choice list. Only one pipeline is currently available:

| pipelineName | pipelinesascode.tekton.dev/pipeline | spec.pipelineRef.name |
|---|---|---|
| `maven-build-ci` | `https://gitlab-gitlab.<ocp-apps-domain>/rhdh/tssc-sample-pipelines/-/raw/main/pipelines/maven-build-ci.yaml` | `maven-build-ci` |

The pipeline reference URL is computed from the cluster's GitLab host (resolved via `provisioning-data`) and the chosen pipeline name. Adding a new target pipeline only requires appending a new entry to the `enum` of the `pipelineName` parameter in `template.yaml`.

## Repository Breakdown

```
quarkus-stssc-pipeline-update-template/
├── skeleton/
│   ├── .tekton/
│   │   └── on-push.yaml   # PipelineRun template (pipeline annotation parameterized)
│   └── Dockerfile          # Dockerfile template (FROM parameterized)
└── template.yaml           # Backstage Template definition
```

## Requirements

* RHDH with the GitLab scaffolder backend module installed (provides `publish:gitlab:merge-request`).
* GitLab integration configured for the `gitlab-gitlab.<ocp-apps-domain>` host.
* The `component:default/provisioning-data` entity present in the catalog.
* GitLab credentials with write access to the target repository.

## Notes / customization

* The skeleton `on-push.yaml` is based on the standard Quarkus/Tekton tssc PipelineRun. If the target repositories use a different PipelineRun shape (different tasks, `pipelineRef`, or `output-image` location), adapt the skeleton file accordingly.
* The action fails if the source branch already exists with an open merge request; use a unique `sourceBranchName` when re-running.