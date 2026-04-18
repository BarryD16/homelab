
# Home Lab Architecture

## Overview

A 8-node Raspberry Pi 5 k3s Kubernetes cluster running home automation, monitoring, and lab services, all accessible externally via Cloudflare Tunnel.

## Hardware

| Device | Model | RAM | Role |
|--------|-------|-----|------|
| node01 | Raspberry Pi 5 | 8GB | Control plane |
| node02 | Raspberry Pi 5 | 8GB | Control plane |
| node03 | Raspberry Pi 5 | 8GB | Control plane |
| worker01 | Raspberry Pi 5 | 8GB | Worker (500GB SSD) |
| worker02 | Raspberry Pi 5 | 8GB | Worker (500GB SSD) |
| worker03 | Raspberry Pi 5 | 8GB | Worker |
| worker04 | Raspberry Pi 5 | 8GB | Worker |
| dashboard01 | Raspberry Pi 5 | 8GB | Worker + Kiosk display |
| RedDisk | AMD PC | - | Local AI server (Ollama/llama3.1) |

## Network

| Device | IP | Notes |
|--------|-----|-------|
| Router (Vodafone Ultra Hub) | 192.168.1.1 | DNS points to AdGuard |
| AdGuard Home | 192.168.1.203 | Network-wide DNS + ad blocking |
| Traefik (ingress) | 192.168.1.200 | Internal ingress controller |
| Grafana | 192.168.1.201 | Monitoring dashboards |
| Portainer | 192.168.1.202 | Container management |
| dashboard01 | 192.168.1.100 | Kiosk display Pi |

## Traffic Flow

### External Access
```
Internet
  └── Cloudflare DNS (orchestratedigital.co.uk)
        └── Cloudflare Tunnel (cloudflared pods in k3s)
              └── Traefik (192.168.1.200)
                    └── Kubernetes Services (ClusterIP)
                          └── Application Pods
```

### Internal DNS
```
Device DNS query
  └── Router (192.168.1.1)
        └── AdGuard Home (192.168.1.203)
              └── Cloudflare DoH (1.1.1.1)
```

### Cluster DNS
```
Pod DNS query
  └── CoreDNS (10.43.0.10)
        └── AdGuard Home ClusterIP (10.43.70.57)
              └── Cloudflare (1.1.1.1 / 8.8.8.8)
```

## Storage

- **Longhorn** — distributed block storage across worker nodes
- **worker01/worker02** — 500GB SSDs for persistent storage
- Persistent Volume Claims used by: AdGuard Home, Home Assistant

## Kubernetes Distribution

- **k3s** — lightweight Kubernetes
- **MetalLB** — LoadBalancer IP allocation (pool: 192.168.1.200-192.168.1.210)
- **Traefik** — ingress controller (built into k3s)
- **cert-manager** — TLS certificate management

## OS

All Raspberry Pi nodes run **Ubuntu Server 24.04 LTS (64-bit)**. Username: `barry`.
