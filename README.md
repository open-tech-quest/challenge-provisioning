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

## Provisioning an Environment

There are two ways to provision a challenge environment: using the Red Hat Demo Platform (RHDP) or bringing your own OpenShift cluster.

### Option 1: Red Hat Demo Platform (RHDP)

This is the recommended approach for Red Hat associates and partners.

1. Open the [Red Hat Demo Platform](https://demo.redhat.com) and order the tile **"Order Field Sourced Content - OpenShift Base"**.
2. Configure the following options:
   - **Create users on cluster?** — Select *Yes* and choose the desired number of users.
   - **Existing Gitops Repo?** — Select *Yes*.
   - **GitOps Repo** — `https://github.com/open-tech-quest/challenge-provisioning.git`
   - **GitOps Path** — The path matching the challenge type you want to deploy:
     | Challenge Type | GitOps Path            |
     |----------------|------------------------|
     | OpenShift      | `challenges/openshift` |
     | RHEL           | `challenges/rhel`      |
     | Ansible        | `challenges/ansible`   |
3. Submit the order. RHDP will provision the cluster, install the GitOps operator, create the users, and deploy the challenge environment automatically via ArgoCD.

### Option 2: Bring Your Own OpenShift Cluster

Use this option when you want to run challenges on an existing OpenShift cluster outside of RHDP.

**Prerequisites:**

- OpenShift **4.22** or later
- An identity provider configured with multiple users (e.g., `user1` through `userN`)
- The **Red Hat OpenShift GitOps Operator** installed on the cluster

**Deploy the challenge:**

Apply an ArgoCD `Application` resource to your cluster. The following example deploys the OpenShift challenge for five users:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: open-tech-quest
  namespace: openshift-gitops
spec:
  destination:
    namespace: open-tech-quest
    server: "https://kubernetes.default.svc"
  project: default
  source:
    repoURL: https://github.com/open-tech-quest/challenge-provisioning.git
    targetRevision: main
    path: challenges/openshift
    helm:
      values: |
        deployer:
          apiUrl: https://api.your-cluster.example.com:6443
          domain: apps.your-cluster.example.com

        multi_user:
          num_users: 5
          users:
            - username: user1
            - username: user2
            - username: user3
            - username: user4
            - username: user5
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Adjust `deployer.apiUrl`, `deployer.domain`, and the `multi_user` section to match your cluster and user setup, then apply:

```bash
oc apply -f application.yaml
```

ArgoCD will pick up the Application and deploy the challenge environment.

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
