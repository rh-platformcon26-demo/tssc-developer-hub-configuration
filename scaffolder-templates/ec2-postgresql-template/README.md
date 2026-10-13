# EC2 PostgreSQL template (GitOps)

RHDH software template that **publishes a GitOps repository** and creates an
**Argo CD Application**. Argo syncs a Tekton `PipelineRun` into the CI namespace;
the pipeline clones [`ansible/ec2-postgresql`](../../../ansible/ec2-postgresql)
and provisions a Fedora EC2 instance with PostgreSQL.

## Flow

1. User fills VPC, subnet, key pair, instance name, and DB password in RHDH.
2. Scaffolder renders `gitops-template/` and publishes it to GitLab
   (`${repoOwner}/${instanceName}-ec2-pg-gitops`).
3. `argocd:create-resources` creates an app-of-apps Application in `tssc-gitops`.
4. That Application syncs `./manifests` (the PipelineRun) into the CI namespace.
5. Tekton runs pipeline `ec2-postgresql`.

## Repository layout (published)

```
catalog-info.yaml
app-of-apps/
  application.yaml      # Argo Application → CI namespace
  kustomization.yaml
manifests/
  pipelinerun.yaml      # fixed-name PipelineRun
  kustomization.yaml
```

## Cluster prerequisites

In the CI namespace (default `tssc-app-ci`):

1. Install Pipeline + Task from `tssc-sample-pipelines`:
   - `pipelines/ec2-postgresql.yaml`
   - `tasks/ansible-ec2-postgresql.yaml`

2. Create Secrets:

```bash
oc -n tssc-app-ci create secret generic aws-credentials \
  --from-literal=AWS_ACCESS_KEY_ID='...' \
  --from-literal=AWS_SECRET_ACCESS_KEY='...'

oc -n tssc-app-ci create secret generic ec2-ssh-key \
  --from-file=ssh-privatekey=~/.ssh/my-keypair.pem
```

## Ansible repository

Default Git URL (from `provisioning-data` labels):

`https://gitlab-gitlab.<ocp-apps-domain>/rhdh/ansible.git`

The playbook path inside that repo must be `ec2-postgresql/`.

## After the run

```bash
oc get pipelinerun -n tssc-app-ci ec2-postgresql-<instanceName>
```

PostgreSQL listens on **localhost only**. Use an SSH tunnel:

```bash
ssh -i my-keypair.pem -L 5432:127.0.0.1:5432 fedora@<ec2_public_ip>
psql -h 127.0.0.1 -p 5432 -U appuser -d appdb
```
