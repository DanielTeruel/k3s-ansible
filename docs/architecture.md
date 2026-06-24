# Arquitectura del Laboratorio

## Diagrama de Red
[Windows Host]

│

└── VMware Workstation (NAT 192.168.1.0/24)

│

├── Control Station  192.168.1.10  (Ansible + kubectl)

├── K3s Master       192.168.1.11  (Control Plane)

└── K3s Worker       192.168.1.12  (Worker Node)

## Flujo de Gestión

Control Station → SSH (key-based) → Master / Worker
Control Station → kubectl → API Server (Master)

## Decisiones Técnicas

| Decisión | Elección | Motivo |
|----------|----------|--------|
| Distribución K8s | K3s | Lightweight, ideal para lab local con recursos limitados |
| Ingress Controller | Traefik (nativo K3s) | Integrado por defecto, sin instalación adicional |
| Storage | local-path provisioner | Nativo K3s, suficiente para lab |
| Automatización | Ansible | Idempotencia, legibilidad YAML, sin agente en nodos |
| Observabilidad | Prometheus + Grafana vía Helm | Stack estándar del sector |

## Sizing de Recursos

| Nodo | vCPU | RAM | Disco |
|------|------|-----|-------|
| Control Station | 2 | 2 GB | 20 GB |
| Master | 2 | 2 GB | 20 GB |
| Worker | 2 | 2 GB | 20 GB |
