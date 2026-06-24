# K3s Cluster con Ansible — Laboratorio de Infraestructura

Aprovisionamiento automatizado de un clúster Kubernetes multi-nodo (K3s) sobre VMware Workstation con Ansible, despliegue de arquitectura 3 capas, autoscaling, self-healing y observabilidad completa con Prometheus y Grafana.

## Arquitectura

| Nodo | IP | Rol |
|------|----|-----|
| Control Station | 192.168.1.10 | Ubuntu Server — Ansible + kubectl |
| Master / Control Plane | 192.168.1.11 | K3s server |
| Worker | 192.168.1.12 | K3s agent |

Red NAT privada bajo VMware Workstation. Gestión push sin contraseña vía SSH + Ansible desde la Control Station.

## Stack Técnico

- **OS:** Ubuntu Server 24.04 LTS
- **IaC / Automatización:** Ansible (Playbooks YAML, `inventory.ini`)
- **Orquestación:** Kubernetes (K3s), Traefik Ingress Controller, local-path provisioner
- **Red:** Netplan (IPs estáticas), UFW, SSH Keys
- **App (namespace `produccion`):** Nginx (frontend), Adminer, MariaDB
- **Observabilidad:** Prometheus + Grafana (vía Helm)
- **Pruebas:** Apache Bench (`ab`), inyección de fallos en endpoints

## Fases del Proyecto

### Fase Previa — IaC con Ansible
- Configuración de red estática con Netplan y firewall UFW
- Aprovisionamiento idempotente de los nodos K3s mediante Ansible Playbooks

### Fase A — Ingress con Traefik
- Ingress Resource para tráfico de monitorización
- Dominio `http://grafana.local` accesible por puerto 80 (sin NodePorts altos)

### Fase B + E — Storage, Secrets y ConfigMaps
- PersistentVolumeClaims con `local-path` provisioner para persistencia de MariaDB
- Kubernetes Secrets para credenciales de DB; ConfigMaps para configuración Nginx
- Validado: MariaDB sobrevive eliminación y recreación de pods

### Fase C — Aplicación 3 Capas (namespace `produccion`)
1. **Frontend:** Nginx declarativo multi-réplica
2. **Gestión DB:** Adminer
3. **Backend:** MariaDB con PVC
- Comunicación interna vía Kubernetes Services

### Fase D — HPA (Autoscaling)
- Horizontal Pod Autoscaler basado en consumo de CPU
- Validado con `ab -n 100000`: escalado automático de 2 a 10 réplicas al 169% de carga

### Fase F — Liveness y Readiness Probes
- Probes en pods Nginx para garantizar resiliencia
- Self-healing validado: inyección de ruta `/error` → fallo de livenessProbe → reinicio automático

### Fase G — Observabilidad y Alertas
- Stack Prometheus + Grafana vía Helm
- Alerta basada en `increase(kube_pod_container_status_restarts_total[5m])`
- Validado: alerta pasa a `Firing` en tiempo real durante pruebas de estrés

## Estructura del Repositorio
k3s-ansible/

├── ansible/

│   ├── inventory/         # inventory.ini con IPs de los nodos

│   ├── playbooks/         # Playbooks de aprovisionamiento

│   └── roles/             # Roles reutilizables

├── k8s/

│   ├── namespace/         # Namespace produccion

│   ├── app/               # Deployments: Nginx, Adminer, MariaDB

│   ├── storage/           # PVC y StorageClass

│   ├── autoscaling/       # HPA

│   ├── probes/            # Liveness y Readiness Probes

│   └── observability/     # Prometheus, Grafana, Ingress

└── docs/

└── architecture.md
