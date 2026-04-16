# Services

## External URLs (via Cloudflare Tunnel)

| Service | URL | Credentials |
|---------|-----|-------------|
| Home Assistant | https://ha.orchestratedigital.co.uk | See Bitwarden |
| Grafana | https://grafana.orchestratedigital.co.uk | See Bitwarden |
| Portainer | https://portainer.orchestratedigital.co.uk | See Bitwarden |
| Uptime Kuma | https://uptime.orchestratedigital.co.uk | See Bitwarden |
| AdGuard Home | https://adguard.orchestratedigital.co.uk | See Bitwarden |
| Homepage | https://home.orchestratedigital.co.uk | No auth |
| Longhorn | https://longhorn.orchestratedigital.co.uk | No auth (internal only) |


## Internal Services

| Service | Internal Address | Port | Notes |
|---------|-----------------|------|-------|
| AdGuard Home | 192.168.1.203 | 80 (web), 53 (DNS) | Network DNS server |
| Grafana | 192.168.1.201 | 80 | Monitoring |
| Portainer | 192.168.1.202 | 9000 | Container management |
| Traefik | 192.168.1.200 | 80/443 | Ingress |
| Home Assistant | ClusterIP only | 8123 | External via tunnel |
| Prometheus | 10.43.199.24 | 9090 | Internal only |
| CoreDNS | 10.43.0.10 | 53 | Cluster DNS |
| AdGuard ClusterIP | 10.43.70.57 | 53 | Used by CoreDNS forwarder |

## Kubernetes Namespaces

| Namespace | Services |
|-----------|----------|
| default | homeassistant, homepage, cloudflared, adguard-home, adguard-exporter |
| monitoring | grafana, prometheus, alertmanager, uptime-kuma, node-exporter |
| portainer | portainer |
| longhorn-system | longhorn storage |
| cert-manager | certificate management |
| kube-system | coredns, metrics-server, traefik |
| metallb-system | metallb load balancer |

## Grafana Dashboards

| Dashboard | ID | Data Source |
|-----------|-----|-------------|
| Node Exporter Full | 1860 | Prometheus |
| AdGuard Home Exporter | 20799 | Prometheus |
| Strava | (imported from plugin) | Strava |

## Grafana Playlist

Dashboard01 kiosk cycles through: Node Exporter Full → AdGuard Home → Strava (1 minute each)

## Home Assistant Integrations

| Integration | Notes |
|-------------|-------|
| Blink | 7 cameras (doorbell, front, front door, dining room, garage, side) |
| Telegram Bot | @BarryHomeAssistantBot, chat ID 6300768342 |

## Automations

| Name | Trigger | Action |
|------|---------|--------|
| Blink Motion - Telegram Alert | Any Blink camera motion | Send Telegram message |

## RedDisk (Local AI Server)

- OS: Ubuntu 25.04
- GPU: AMD RX 7600 (ROCm, HSA_OVERRIDE_GFX_VERSION=11.0.0)
- Ollama running llama3.1 8B with GPU acceleration
- OpenClaw connected to Telegram
