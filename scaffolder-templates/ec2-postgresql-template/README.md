# EC2 PostgreSQL template (Pipelines-as-Code)

Same pattern as the Quarkus TSSC template:

1. **Source repo** with `.tekton/on-push.yaml` (PaC PipelineRun)
2. **GitOps repo** with PaC `Repository` + GitLab webhook Job
3. **Argo CD** app-of-apps syncs only the CI overlay into `tssc-app-ci`

## Flow

1. User fills VPC, subnet, key pair, name, and DB password in RHDH.
2. Scaffolder publishes:
   - `${repoOwner}/${name}` — source + `.tekton/on-push.yaml` + catalog
   - `${repoOwner}/${name}-gitops` — Repository CR, webhook Job, app-of-apps
3. Argo CD syncs into `tssc-app-ci`:
   - `Repository` pointing at the source repo
   - Job that creates the GitLab push webhook and a dummy commit
4. Pipelines-as-Code creates the PipelineRun from `.tekton/` (Pipeline + Task
   are vendored in the source repo — they are not yet present on the lab
   GitLab mirror of `tssc-sample-pipelines`, which returned 404 for remote URLs).

## Repository layout

**Source**

```
.tekton/on-push.yaml       # PipelineRun (PaC trigger)
.tekton/pipeline.yaml      # Pipeline: init → provision → summary
.tekton/task-init.yaml     # Task ec2-postgresql-init
.tekton/task.yaml          # Task ansible-ec2-postgresql
.tekton/task-summary.yaml  # Task ec2-postgresql-summary
catalog-info.yaml          # System, Component, EC2 + DB Resources
README.md
```

**GitOps**

```
catalog-info.yaml          # gitops Resource
app-of-apps/
  ci-tekton.yaml
  kustomization.yaml
components/<name>/overlays/ci/
  source-repository.yaml
  secret-gitlab-webhook.yaml
  cm-create-webhook.yaml
  job-create-webhook.yaml
  kustomization.yaml
```

## Catalog / RHDH visibility

Entities use:

- `janus-idp.io/tekton: <name>`
- `backstage.io/kubernetes-id: <name>`

## Cluster prerequisites

In **tssc-app-ci**:

```bash
oc -n tssc-app-ci create secret generic aws-credentials \
  --from-literal=AWS_ACCESS_KEY_ID='...' \
  --from-literal=AWS_SECRET_ACCESS_KEY='...'

oc -n tssc-app-ci create secret generic ec2-ssh-key \
  --from-file=ssh-privatekey=~/.ssh/my-keypair.pem
```

`gitlab-auth-secret` is expected to already exist (TSSC lab default).

## After the run

```bash
oc get repository,job,pipelinerun -n tssc-app-ci -l janus-idp.io/tekton=<name>
```

PostgreSQL listens on **localhost only**. SSH tunnel:

```bash
ssh -i my-keypair.pem -L 5432:127.0.0.1:5432 fedora@<ec2_public_ip>
psql -h 127.0.0.1 -p 5432 -U appuser -d appdb
```
