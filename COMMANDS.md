# Home Lab Command Reference

## kubectl - General

| Command | Description |
|---------|-------------|
| `kubectl get nodes` | List all nodes and their status |
| `kubectl get nodes -o wide` | List nodes with IP and role detail |
| `kubectl get pods -A` | List all pods across all namespaces |
| `kubectl get pods -n <namespace>` | List pods in a specific namespace |
| `kubectl get pods -n <namespace> -w` | Watch pods in real time |
| `kubectl get pods -A \| grep -i <name>` | Search for a pod by name |
| `kubectl get services -A` | List all services across all namespaces |
| `kubectl get pvc -A` | List all persistent volume claims |
| `kubectl describe pod <pod> -n <namespace>` | Detailed info about a pod including events |
| `kubectl logs <pod> -n <namespace>` | View pod logs |
| `kubectl logs <pod> -n <namespace> --tail=50` | View last 50 lines of pod logs |
| `kubectl delete pod <pod> -n <namespace> --force` | Force delete a stuck pod |

---

## kubectl - Deployments

| Command | Description |
|---------|-------------|
| `kubectl apply -f <file>.yaml` | Apply a manifest file |
| `kubectl delete -f <file>.yaml` | Delete resources defined in a manifest |
| `kubectl rollout restart deployment/<name> -n <namespace>` | Restart a deployment |
| `kubectl rollout status deployment/<name> -n <namespace>` | Check rollout status |
| `kubectl scale deployment/<name> -n <namespace> --replicas=0` | Scale down a deployment |
| `kubectl scale deployment/<name> -n <namespace> --replicas=1` | Scale up a deployment |
| `kubectl edit deployment/<name> -n <namespace>` | Edit a deployment in vi |
| `kubectl patch deployment <name> -n <namespace> --type=json -p='[...]'` | Patch a deployment field |

---

## kubectl - Exec & Access

| Command | Description |
|---------|-------------|
| `kubectl exec -it <pod> -n <namespace> -- bash` | Open bash shell in a pod |
| `kubectl exec -it <pod> -n <namespace> -- sh` | Open sh shell in a pod (Alpine) |
| `kubectl exec -it <pod> -n <namespace> -- <command>` | Run a single command in a pod |

### Useful pod selectors
```bash
# Get pod name dynamically by label
kubectl get pod -n <namespace> -l app=<name> -o jsonpath='{.items[0].metadata.name}'

# Exec using dynamic pod name
kubectl exec -it $(kubectl get pod -n default -l app=homeassistant -o jsonpath='{.items[0].metadata.name}') -n default -- bash
```

---

## kubectl - ConfigMaps

| Command | Description |
|---------|-------------|
| `kubectl get configmap <name> -n <namespace> -o yaml` | View a configmap |
| `kubectl edit configmap <name> -n <namespace>` | Edit a configmap in vi |

---

## k3s Cluster Management

| Command | Description |
|---------|-------------|
| `sudo cat /var/lib/rancher/k3s/server/node-token` | Get join token for adding worker nodes |
| `kubectl get nodes` | Verify all nodes are Ready |
| `kubectl label node <node> node-role.kubernetes.io/worker=worker` | Label a node as worker |
| `curl -sfL https://get.k3s.io \| K3S_URL=https://192.168.1.2:6443 K3S_TOKEN=<token> sh -s - agent` | Join a node to the cluster |

---

## Helm

| Command | Description |
|---------|-------------|
| `helm list -A` | List all Helm releases |
| `helm get values <release> -n <namespace>` | View current values for a release |
| `helm upgrade <release> <chart> -n <namespace> --reuse-values` | Upgrade keeping existing values |
| `helm upgrade <release> <chart> -n <namespace> --set <key>=<value> --reuse-values` | Upgrade with a new value |

---

## CoreDNS

| Command | Description |
|---------|-------------|
| `kubectl get configmap coredns -n kube-system -o yaml` | View CoreDNS config |
| `kubectl edit configmap coredns -n kube-system` | Edit CoreDNS config |
| `kubectl rollout restart deployment/coredns -n kube-system` | Restart CoreDNS |

---

## Home Assistant

| Command | Description |
|---------|-------------|
| `kubectl get pods -n default \| grep homeassistant` | Find HA pod |
| `kubectl rollout restart deployment/homeassistant -n default` | Restart Home Assistant |
| `kubectl exec -it <ha-pod> -n default -- bash` | Shell into HA container |
| `vi /config/configuration.yaml` | Edit HA config (inside pod) |

---

## AdGuard Home

| Command | Description |
|---------|-------------|
| `kubectl get pods -n default \| grep adguard` | Find AdGuard pod |
| `kubectl rollout restart deployment/adguard-home -n default` | Restart AdGuard |
| `curl http://192.168.1.203/metrics` | Check AdGuard metrics endpoint |
| `kubectl scale deployment adguard-home -n default --replicas=0` | Stop AdGuard (for config editing) |

---

## Longhorn Storage

| Command | Description |
|---------|-------------|
| `kubectl get pvc -A` | List all persistent volume claims |
| `kubectl get pv` | List all persistent volumes |
| `kubectl describe pvc <name> -n <namespace>` | Check PVC status and events |

---

## Prometheus & Grafana

| Command | Description |
|---------|-------------|
| `kubectl get pods -n monitoring` | List monitoring pods |
| `kubectl rollout restart deployment/kube-prometheus-stack-grafana -n monitoring` | Restart Grafana |
| `curl -s http://10.43.199.24:9090/api/v1/targets \| python3 -m json.tool \| grep -i <name>` | Check Prometheus scrape targets |
| `curl -s "http://10.43.199.24:9090/api/v1/targets?state=active"` | List active Prometheus targets |

---

## Cloudflare Tunnel

| Command | Description |
|---------|-------------|
| `kubectl get pods -n default \| grep cloudflared` | Find tunnel pods |
| `kubectl logs -n default <cloudflared-pod> --tail=20` | Check tunnel logs |

---

## dashboard01 Kiosk

| Command | Description |
|---------|-------------|
| `ssh barry@192.168.1.100` | SSH into dashboard01 |
| `sudo loginctl terminate-user barry` | Restart kiosk session |
| `nano ~/.bash_profile` | Edit kiosk startup config |
| `sudo systemctl status x11.service` | Check Wayfire/kiosk service |
| `sudo reboot` | Reboot dashboard01 |

---

## Node SSH Access

| Command | Description |
|---------|-------------|
| `ssh barry@192.168.1.2` | node01 (control plane) |
| `ssh barry@192.168.1.153` | node02 (control plane) |
| `ssh barry@192.168.1.131` | node03 (control plane) |
| `ssh barry@192.168.1.74` | worker01 |
| `ssh barry@192.168.1.95` | worker02 |
| `ssh barry@192.168.1.3` | worker03 |
| `ssh barry@192.168.1.47` | worker04 |
| `ssh barry@192.168.1.100` | dashboard01 |

---

## Git (homelab repo)

| Command | Description |
|---------|-------------|
| `cd ~/homelab` | Navigate to homelab repo |
| `git add .` | Stage all changes |
| `git commit -m "message"` | Commit changes |
| `git push` | Push to GitHub |
| `git pull` | Pull latest from GitHub |

---

## General Linux

| Command | Description |
|---------|-------------|
| `sudo apt update && sudo apt upgrade -y` | Update packages |
| `sudo reboot` | Reboot node |
| `sudo systemctl status <service>` | Check service status |
| `sudo systemctl restart <service>` | Restart a service |
| `sudo journalctl -u <service> -n 50` | View service logs |
| `ip a` | Show network interfaces and IPs |
| `ping <host>` | Test connectivity |
| `nslookup <domain>` | DNS lookup |
| `curl -s https://<url>` | Test HTTP endpoint |

