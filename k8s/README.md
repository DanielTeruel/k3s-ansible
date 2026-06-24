# Manifiestos Kubernetes

Manifiestos declarativos organizados por funcionalidad.

## Orden de aplicación

```bash
kubectl apply -f namespace/
kubectl apply -f storage/
kubectl apply -f app/
kubectl apply -f autoscaling/
kubectl apply -f probes/
kubectl apply -f observability/
```

## Notas

- Namespace: `produccion`
- Ingress Controller: Traefik (nativo K3s)
- Storage: `local-path` provisioner (nativo K3s)
- Grafana en `http://grafana.local` (añadir en `C:\Windows\System32\drivers\etc\hosts` del anfitrión)
