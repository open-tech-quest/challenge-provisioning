# Agent Instructions

This repository provisions the Open Tech Quest challenge infrastructure on Red Hat Demo Platform (RHDP) using a GitOps approach. ArgoCD deploys Helm charts, and Ansible handles complex orchestration tasks (operator installs, secret generation, wait-for-ready logic).

Based on the [field-sourced-content-template](https://github.com/rhpds/field-sourced-content-template).

## Repository Layout

```
challenge-provisioning/
├── challenges/              # One chart per challenge type
│   ├── openshift/           # OpenShift challenge — Ansible-in-a-Job pattern
│   │   ├── Chart.yaml
│   │   ├── values.yaml      # Job config, repo URL, collections, extraVars
│   │   ├── templates/       # K8s Job, RBAC, ConfigMap templates
│   │   └── playbooks/       # Ansible content
│   │       ├── site.yml     # Main playbook
│   │       └── roles/       # Reusable roles (operator-install, deploy-showroom, etc.)
│   ├── rhel/                # (planned) RHEL challenge environment
│   └── ansible/             # (planned) Ansible challenge environment
├── examples/
│   └── helm/                # Reference — Helm App-of-Apps pattern
│       ├── Chart.yaml
│       ├── values.yaml      # Central config — enable/disable components
│       ├── templates/       # ArgoCD Application generator
│       └── components/      # Standalone Helm sub-charts (operator, hello-world, showroom)
├── roles/
│   └── ocp4_workload_field_content/  # Reference-only AgnosticD workload role (DO NOT MODIFY)
└── docs/                    # Developer guides, architecture diagrams, update specs
```

## Key Conventions

1. **Each challenge type lives under `challenges/<type>/`** — currently OpenShift is active; RHEL and Ansible are planned
2. **Helm charts are the deployment unit** — even Ansible playbooks are wrapped in a Helm chart that creates a K8s Job
3. **ArgoCD sync waves** control execution order (`syncWave` in values.yaml)
4. **RHDP labels are required** for platform integration:
   - `demo.redhat.com/application: "<name>"` — health monitoring
   - `demo.redhat.com/userinfo: ""` — pass data (URLs, credentials) back to AgnosticD/RHDP UI
5. **AgnosticD variable prefix**: all workload variables use `ocp4_workload_` (e.g., `ocp4_workload_field_content_gitops_repo_url`)
6. **`roles/ocp4_workload_field_content/` is reference-only** — the canonical role lives in `agnosticd/core_workloads`; never modify it here
7. **Deployer values are auto-injected** by the workload role:
   - `deployer.domain` — cluster apps domain (e.g., `apps.cluster-guid.sandbox.opentlc.com`)
   - `deployer.apiUrl` — cluster API URL
8. **Ansible playbooks must be idempotent** — safe to run multiple times via `state: present`
9. **Pin collection versions** in `values.yaml` or `requirements.yml` for reproducible builds

## Testing

```bash
# OpenShift challenge: run playbook directly against a cluster
cd challenges/openshift/playbooks
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml -e "cluster_domain=..." -e "namespace=test"

# Helm (reference): lint and template locally
helm lint examples/helm/
helm template my-release examples/helm/ --set deployer.domain=apps.cluster.example.com
```

## Available Skills

- **add-component**: Add a new Helm component or Ansible role to the provisioning (operator, application, service)
- **review-provisioning**: Review and validate Helm charts and Ansible playbooks for correctness and RHDP compliance
