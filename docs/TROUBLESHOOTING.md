# Troubleshooting Log

A record of issues encountered and their fixes.

---

## CoreDNS crash loop after AdGuard deployment

**Symptom:** CoreDNS enters CrashLoopBackOff, all external DNS fails across the cluster.

**Cause:** When AdGuard was deployed and set as the network DNS server, the node's `/etc/resolv.conf` started pointing at AdGuard (192.168.1.203). CoreDNS was configured to forward DNS via `/etc/resolv.conf`, but AdGuard wasn't reachable from inside the cluster network. Additionally, a config syntax error (missing closing `}`) caused CoreDNS to fail to parse its Corefile.

**Fix:**
1. Edit CoreDNS configmap: `kubectl edit configmap coredns -n kube-system`
2. Change `forward . /etc/resolv.conf` to `forward . 10.43.70.57` (AdGuard ClusterIP)
3. Ensure the `.:53 { }` block has its closing brace
4. Restart CoreDNS: `kubectl rollout restart deployment/coredns -n kube-system`

---

## Home Assistant pod DNS not working

**Symptom:** HA pod cannot resolve external hostnames. Telegram, Blink, and other integrations fail with DNS errors.

**Cause:** The HA deployment had `dnsPolicy: None` with hardcoded nameservers (1.1.1.1, 8.8.8.8). After AdGuard was deployed and bound port 53, iptables blocked direct UDP port 53 traffic from pods to external IPs.

**Fix:**
```bash
kubectl patch deployment homeassistant -n default --type=json \
  -p='[{"op": "remove", "path": "/spec/template/spec/dnsConfig"}, 
       {"op": "replace", "path": "/spec/template/spec/dnsPolicy", "value": "ClusterFirst"}]'
```

---

## Blink integration rate limiting

**Symptom:** Blink shows as "Unavailable" in HA, cycling between armed and unavailable every 5 minutes. Re-authentication fails with "Unable to extract region from response None".

**Cause:** HA was polling Blink's cloud API too frequently, causing temporary rate limiting/session drops.

**Fix:** Wait 20-30 minutes for the rate limit to clear, then re-add the Blink integration fresh via Settings → Devices & Services → Add Integration → Blink. Do not attempt to re-authenticate multiple times in quick succession.

**Note:** The `blink: scan_interval:` YAML config is no longer supported in newer HA versions. Use the UI only.

---

## Telegram bot "All connection attempts failed"

**Symptom:** HA cannot connect to Telegram API. `telegram_bot.send_message` action fails.

**Cause:** DNS resolution failure inside the HA pod (see HA pod DNS fix above), or the `target` parameter being used in the send_message action.

**Fix:**
1. Fix DNS (see above)
2. Do not use the `target` parameter in `telegram_bot.send_message` — it has been deprecated and removed

Correct action:
```yaml
action: telegram_bot.send_message
data:
  message: "Your message here"
```

---

## AdGuard config wiped on pod restart

**Symptom:** Changes made to AdGuardHome.yaml inside the pod are lost after restart.

**Cause:** Original deployment used `emptyDir` volumes which do not persist across pod restarts.

**Fix:** Redeploy AdGuard with Longhorn PVCs. See `manifests/adguard.yaml`. When AdGuard is scaled down for config editing, use a temporary Alpine pod to mount and edit the PVC directly:

```bash
kubectl scale deployment adguard-home -n default --replicas=0
kubectl run adguard-edit --rm -it --image=alpine --restart=Never -n default \
  --overrides='{"spec":{"volumes":[{"name":"conf","persistentVolumeClaim":{"claimName":"adguard-conf"}}],"containers":[{"name":"adguard-edit","image":"alpine","stdin":true,"tty":true,"command":["sh"],"volumeMounts":[{"name":"conf","mountPath":"/conf"}]}]}}'
# edit /conf/AdGuardHome.yaml, then exit
kubectl scale deployment adguard-home -n default --replicas=1
```

---

## AdGuard RWO volume multi-attach error

**Symptom:** New AdGuard pod stuck in ContainerCreating with "Multi-Attach error — volume already used by pod".

**Cause:** ReadWriteOnce (RWO) Longhorn volumes can only be attached to one pod at a time. During a rolling restart, both old and new pods exist briefly and both try to claim the volume.

**Fix:** Force delete the old pod to release the volume:
```bash
kubectl delete pod -n default <old-pod-name> --force
```

---

## AdGuard Prometheus metrics not scraped

**Symptom:** `adguard_num_dns_queries` returns no data in Grafana despite exporter running.

**Cause 1:** ServiceMonitor was in `default` namespace but Prometheus only watches `monitoring` namespace.
**Fix:** Move ServiceMonitor to `monitoring` namespace and add `namespaceSelector` to match `default`.

**Cause 2:** Prometheus dropping targets due to scheme defaulting to HTTPS.
**Fix:** Add `scheme: http` to ServiceMonitor endpoints.

**Cause 3 (final fix):** Use `additionalScrapeConfigs` in Helm values instead of ServiceMonitor:
```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring \
  --set prometheus.prometheusSpec.additionalScrapeConfigs[0].job_name=adguard \
  --set prometheus.prometheusSpec.additionalScrapeConfigs[0].static_configs[0].targets[0]="<adguard-exporter-clusterip>:9617" \
  --reuse-values
```

---

## Dashboard01 kiosk display not starting

**Symptom:** Kiosk fails to launch, showing "Failed to spawn client" or X11/Wayland errors.

**Cause:** Pi 5 requires Wayland (cage compositor) rather than X11. Snap version of Chromium conflicts with cage due to AppArmor restrictions.

**Fix:**
1. Use `cage` as the Wayland compositor
2. Install non-snap `chromium-browser` or use Firefox (`sudo apt install -y firefox`)
3. Set correct environment variables in `~/.bash_profile`:

```bash
if [ -z "$WAYLAND_DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
    export WLR_DRM_NO_ATOMIC=1
    export WLR_NO_HARDWARE_CURSORS=1
    export WLR_DRM_DEVICES=/dev/dri/card1
    cage -- firefox --kiosk "<your-grafana-url>&kiosk"
fi
```

4. Enable autologin to tty1:
```bash
sudo mkdir -p /etc/systemd/system/getty@tty1.service.d
sudo nano /etc/systemd/system/getty@tty1.service.d/autologin.conf
```
```ini
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin barry --noclear %I $TERM
```

---

## Grafana password reset after Helm upgrade

**Symptom:** Grafana admin password stops working after a `helm upgrade`.

**Fix:** Retrieve the current auto-generated password:
```bash
kubectl get secret --namespace monitoring kube-prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
Then change it in Grafana UI: Profile → Change Password.

---

## vi tips (common mistakes)

- To exit insert mode: press `Esc` (or `Ctrl+[`)
- To save and quit: `Esc` then `:wq` then `Enter`
- To quit without saving: `Esc` then `:q!` then `Enter`
- To delete a line: navigate to line, press `dd`
- To go to end of file: press `G`
- To open new line below: press `o` (enters insert mode)
- **Never type `:wq` while still in insert mode** — it will be inserted into the file

## CoreDNS cannot resolve external domains (SERVFAIL)

**Symptom:** Pods get SERVFAIL when resolving external hostnames. ArgoCD shows `server misbehaving` errors when trying to reach GitHub.

**Cause:** CoreDNS forwards to `/etc/resolv.conf` which on Ubuntu with systemd-resolved points to `127.0.0.53` — a loopback address not reachable from within pods.

**Fix:** Edit the CoreDNS configmap and replace the forward directive:
```bash
kubectl edit configmap coredns -n kube-system
```
Change:

eof

## CoreDNS cannot resolve external domains (SERVFAIL)

**Symptom:** Pods get SERVFAIL when resolving external hostnames. ArgoCD shows `server misbehaving` errors when trying to reach GitHub.

**Cause:** CoreDNS forwards to `/etc/resolv.conf` which on Ubuntu with systemd-resolved points to `127.0.0.53` — a loopback address not reachable from within pods.

**Fix:** Edit the CoreDNS configmap and replace the forward directive:
```bash
kubectl edit configmap coredns -n kube-system
```
Change:To:Then restart CoreDNS:
```bash
kubectl rollout restart deployment coredns -n kube-system
```

**Note:** This change is not persisted in Git as the coredns configmap is managed by k3s. It will need to be reapplied if the cluster is rebuilt.
