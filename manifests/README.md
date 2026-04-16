# Manifests

This folder contains Kubernetes manifests for all deployed services.

## Files

| File | Description |
|------|-------------|
| adguard.yaml | AdGuard Home deployment with Longhorn PVCs |
| adguard-exporter.yaml | AdGuard Prometheus exporter |
| adguard-servicemonitor.yaml | ServiceMonitor for AdGuard exporter |

## Notes

- All manifests are applied with `kubectl apply -f <file>.yaml`
- Longhorn must be running before applying any PVC-dependent manifests
- MetalLB must be configured with IP pool 192.168.1.200-210 before LoadBalancer services will get IPs
