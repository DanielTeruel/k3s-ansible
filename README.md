# K3s Cluster with Ansible — Infrastructure Lab

Automated provisioning of a multi-node Kubernetes cluster (K3s) on VMware Workstation using Ansible, 
3-tier application deployment, autoscaling, self-healing and full observability with Prometheus and Grafana.

## Architecture

| Node | IP | Role |
|------|----|------|
| Control Station | 192.168.1.10 | Ubuntu Server — Ansible + kubectl |
| Master / Control Plane | 192.168.1.11 | K3s server |
| Worker | 192.168.1.12 | K3s agent |

Private NAT network under VMware Workstation. Passwordless push management via SSH + Ansible from the Control Station.

## Tech Stack

- **OS:** Ubuntu Server 24.04 LTS
- **IaC / Automation:** Ansible (YAML Playbooks, `inventory.ini`)
- **Orchestration:** Kubernetes (K3s), Traefik Ingress Controller, local-path provisioner
- **Networking:** Netplan (static IPs), UFW, SSH Keys
- **App (namespace `produccion`):** Nginx (frontend), Adminer, MariaDB
- **Observability:** Prometheus + Grafana (via Helm)
- **Load Testing:** Apache Bench (`ab`), endpoint fault injection

## Project Phases

### Pre-Phase — IaC with Ansible
- Static network configuration with Netplan and UFW firewall
- Idempotent automated provisioning of K3s nodes via Ansible Playbooks

### Phase A — Ingress with Traefik
- Ingress Resource for monitoring traffic routing
- `http://grafana.local` accessible on port 80 — no high NodePorts

### Phase B + E — Storage, Secrets and ConfigMaps
- PersistentVolumeClaims with `local-path` provisioner for MariaDB data persistence
- Kubernetes Secrets for DB credentials; ConfigMaps for Nginx static config
- Validated: MariaDB survives pod deletion and recreation

### Phase C — 3-Tier Application (namespace `produccion`)
1. **Frontend:** Declarative multi-replica Nginx
2. **DB Management:** Adminer
3. **Backend:** MariaDB with PVC
- Internal communication via Kubernetes Services

### Phase D — HPA (Autoscaling)
- Horizontal Pod Autoscaler based on CPU consumption
- Validated with `ab -n 100000`: automatic scale-out from 2 to 10 replicas at 169% CPU load

### Phase F — Liveness and Readiness Probes
- Health probes on Nginx pods for application resilience
- Self-healing validated: injecting `/error` route → livenessProbe failure → automatic pod restart by Kubernetes

### Phase G — Observability and Alerting
- Full Prometheus + Grafana stack via Helm
- Alert rule based on `increase(kube_pod_container_status_restarts_total[5m])`
- Validated: alert transitions to `Firing` in real time during stress tests

## Repository Structure
k3s-ansible/

├── ansible/

│   ├── inventory/         # inventory.ini with node IPs

│   ├── playbooks/         # Provisioning playbooks

│   └── roles/             # Reusable roles

├── k8s/

│   ├── namespace/         # produccion namespace definition

│   ├── app/               # Deployments: Nginx, Adminer, MariaDB

│   ├── storage/           # PVC and StorageClass

│   ├── autoscaling/       # HPA

│   ├── probes/            # Liveness and Readiness Probes

│   └── observability/     # Prometheus, Grafana, Ingress

└── docs/

└── architecture.md    # Diagram and technical decisions

## Quick Start

```bash
# 1. Provision network and firewall on all nodes
ansible-playbook -i ansible/inventory/inventory.ini ansible/playbooks/setup-network.yml

# 2. Install K3s Control Plane
ansible-playbook -i ansible/inventory/inventory.ini ansible/playbooks/install-k3s-master.yml

# 3. Join Worker node
ansible-playbook -i ansible/inventory/inventory.ini ansible/playbooks/install-k3s-worker.yml

# 4. Deploy application stack
kubectl apply -f k8s/namespace/
kubectl apply -f k8s/storage/
kubectl apply -f k8s/app/
kubectl apply -f k8s/autoscaling/
kubectl apply -f k8s/observability/
```

## Certifications & Context

This lab was built as a hands-on complement to the **Microsoft Azure Administrator (AZ-104)** certification,
covering Kubernetes orchestration, infrastructure-as-code, observability patterns and GitOps principles
in a local on-premise environment before applying them in cloud-native Azure workloads.
