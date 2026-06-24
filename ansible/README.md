# Ansible — Aprovisionamiento del Clúster K3s

Playbooks para configuración de red, firewall e instalación idempotente de K3s en los nodos.

## Inventario (`inventory/inventory.ini`)

```ini
[control_plane]
master ansible_host=192.168.1.11

[workers]
worker1 ansible_host=192.168.1.12

[all:vars]
ansible_user=daniel
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

## Playbooks

| Playbook | Descripción |
|----------|-------------|
| `playbooks/setup-network.yml` | Configura IPs estáticas con Netplan y reglas UFW |
| `playbooks/install-k3s-master.yml` | Instala K3s server en el Control Plane |
| `playbooks/install-k3s-worker.yml` | Une el Worker al clúster via token |

## Ejecución

```bash
ansible-playbook -i inventory/inventory.ini playbooks/setup-network.yml
ansible-playbook -i inventory/inventory.ini playbooks/install-k3s-master.yml
ansible-playbook -i inventory/inventory.ini playbooks/install-k3s-worker.yml
```
