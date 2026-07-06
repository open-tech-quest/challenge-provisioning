# OpenShift Challenge Environment

Provisions the OpenShift challenge environment using Ansible playbooks executed via a Kubernetes Job.

## Architecture

```
challenges/openshift/
├── Chart.yaml
├── values.yaml           # Configuration for Job, collections, extraVars
├── templates/            # Job, RBAC, ConfigMap templates
└── playbooks/            # Ansible content
    ├── site.yml          # Main playbook with workflow
    └── roles/            # Reusable roles (operator-install, deploy-showroom, etc.)
```

ArgoCD deploys this as a Helm chart. The chart creates a Kubernetes Job that clones the playbooks and runs Ansible.

## What Gets Deployed

| Component | Description |
|-----------|-------------|
| Web Terminal | Operator installed via OLM with wait-for-ready |
| SSH Keypair | Generated and stored as Secret |
| Hello World | Sample httpd application |
| Showroom | Lab guide with terminal |

## Configuration

All settings are in `values.yaml`. Key sections:

```yaml
ansible:
  repository:
    url: "https://github.com/open-tech-quest/challenge-provisioning"
    path: "challenges/openshift/playbooks"
  playbook: "site.yml"
  extraVars:
    operator_name: "web-terminal"
    showroom_git_repo: "https://github.com/your-org/your-lab.git"
```

See comments in [values.yaml](values.yaml) and [playbooks/site.yml](playbooks/site.yml) for detailed documentation.

## Testing Locally

```bash
cd playbooks
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml \
  -e "cluster_domain=$(oc get ingresses.config.openshift.io cluster -o jsonpath='{.spec.domain}')" \
  -e "namespace=test"
```

## Troubleshooting

```bash
oc get jobs -n ansible-runner
oc logs -n ansible-runner job/openshift-challenge
```
