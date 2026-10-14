# ${{ values.name }}

Pipelines-as-Code definition to provision a Fedora EC2 instance with PostgreSQL
via the `ec2-postgresql` Tekton pipeline (Ansible).

## Trigger

Push to `master` runs `.tekton/on-push.yaml` with the local Pipeline/Task
definitions in `.tekton/pipeline.yaml` and `.tekton/task.yaml`.

## Prerequisites (CI namespace)

- Secret `aws-credentials` (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
- Secret `ec2-ssh-key` (`ssh-privatekey`)
