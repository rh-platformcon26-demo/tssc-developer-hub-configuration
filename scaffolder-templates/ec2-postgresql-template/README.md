# EC2 PostgreSQL template

RHDH software template that creates a **one-shot** OpenShift Pipelines `PipelineRun`
to provision a Fedora EC2 instance with PostgreSQL using the
[`ansible/ec2-postgresql`](../../../ansible/ec2-postgresql) playbook.

## Flow

1. User fills VPC, subnet, key pair, instance name, and DB password in RHDH.
2. Scaffolder applies a `PipelineRun` in the CI namespace (`kubernetes:apply`).
3. The `ec2-postgresql` pipeline clones the ansible repo and runs `playbooks/site.yml`.

## Cluster prerequisites

Install (same namespace as the PipelineRun, default `tssc-app-ci`):

- Pipeline + Task from `tssc-sample-pipelines`:
  - `pipelines/ec2-postgresql.yaml`
  - `tasks/ansible-ec2-postgresql.yaml`

Create Secrets (you manage these):

```bash
# AWS credentials
oc -n tssc-app-ci create secret generic aws-credentials \
  --from-literal=AWS_ACCESS_KEY_ID='...' \
  --from-literal=AWS_SECRET_ACCESS_KEY='...'

# EC2 SSH private key (PEM for the key pair named in the form)
oc -n tssc-app-ci create secret generic ec2-ssh-key \
  --from-file=ssh-privatekey=~/.ssh/my-keypair.pem
```

## Ansible repository

Default Git URL (from `provisioning-data` labels):

`https://gitlab-gitlab.<ocp-apps-domain>/rhdh/ansible.git`

The playbook path inside that repo must be `ec2-postgresql/`.

## After the run

PostgreSQL listens on **localhost only**. Use an SSH tunnel:

```bash
ssh -i my-keypair.pem -L 5432:127.0.0.1:5432 fedora@<ec2_public_ip>
psql -h 127.0.0.1 -p 5432 -U appuser -d appdb
```
