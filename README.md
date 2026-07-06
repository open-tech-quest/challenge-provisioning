# Challenge Provisioning

Provisions the Open Tech Quest challenge infrastructure on Red Hat Demo Platform (RHDP) using a GitOps approach.

Based on the [field-sourced-content-template](https://github.com/rhpds/field-sourced-content-template).

## Overview

ArgoCD deploys Helm charts to provision challenge environments. Each challenge type has its own provisioning chart under `challenges/`. Ansible handles complex orchestration (operator installs, secret generation, wait-for-ready logic) via Kubernetes Jobs wrapped in Helm charts.

## Challenge Types

| Type | Location | Status |
|------|----------|--------|
| OpenShift | [challenges/openshift/](challenges/openshift/) | Active |
| RHEL | `challenges/rhel/` | Planned |
| Ansible | `challenges/ansible/` | Planned |

## RHDP Integration

Label resources for platform integration:

```yaml
# Health monitoring
metadata:
  labels:
    demo.redhat.com/application: "my-demo"

# Pass data back to AgnosticD (URLs, credentials, etc.)
metadata:
  labels:
    demo.redhat.com/userinfo: ""
```

## Documentation

- [challenges/openshift/README.md](challenges/openshift/README.md) - OpenShift challenge provisioning
- [examples/helm/README.md](examples/helm/README.md) - Helm deployment reference
- [docs/ansible-developer-guide.md](docs/ansible-developer-guide.md) - In-depth Ansible patterns
- [docs/SHOWROOM-UPDATE-SPEC.md](docs/SHOWROOM-UPDATE-SPEC.md) - Showroom maintenance guide

## Repository Structure

```
challenge-provisioning/
├── challenges/
│   ├── openshift/       # OpenShift challenge environment
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── templates/   # K8s Job, RBAC, ConfigMap templates
│   │   └── playbooks/   # Ansible content and roles
│   ├── rhel/            # (planned) RHEL challenge environment
│   └── ansible/         # (planned) Ansible challenge environment
├── examples/
│   └── helm/            # Helm App-of-Apps reference pattern
├── roles/
│   └── ocp4_workload_field_content/  # AgnosticD workload role (reference-only)
└── docs/                # Developer guides and diagrams
```
