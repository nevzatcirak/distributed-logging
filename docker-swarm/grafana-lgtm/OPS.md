# Swarm Hardened LGTM Stack — Single Node OPS Runbook

This runbook describes how to deploy and operate the hardened **Grafana LGTM stack**
(Grafana, Prometheus, Loki, Tempo, Promtail, OpenTelemetry Collector, Alertmanager)
on a **single-node Docker Swarm**.

In this setup:
- The Swarm manager is the only node
- All services run on this node
- The architecture remains Swarm-native and can later be expanded to multi-node
  without changing stack semantics

---

## Stack Behavior Summary

- Single-node Swarm (manager == worker)
- Grafana dashboards are bind-mounted from `/dashboards` (read-only)
- Prometheus loads alert rules from `/etc/prometheus/rules/*.yml`
- Prometheus scrapes **only** services labeled `metrics=prometheus`
- Promtail collects logs **only** from containers labeled `logging=promtail`
- Docker socket is **never** mounted directly (docker-socket-proxy is used)
- Only Grafana is exposed externally (port 3000)

---

## 0) Prerequisites

- Linux host (VM or bare metal)
- Docker Engine installed
- Minimum recommended resources:
  - CPU: 4 cores
  - RAM: 8 GB
  - Disk: SSD preferred (for Loki / Tempo)

---

## 1) Initialize Single-Node Swarm

```bash
docker swarm init
```

Verify Swarm mode:

```bash
docker info | grep -i swarm
docker node ls
```

---

## 2) Label the Node

```bash
docker node update --label-add observability=true $(docker node ls -q)
```

---

## 3) Prepare Host Paths

```bash
sudo mkdir -p ./observability/{prometheus,loki,tempo,alertmanager,promtail}
```

---

## 4) Permissions

```bash
sudo chmod -R 755 ./dashboards
```

---

## 5) Create Secrets

```bash
printf '%s' 'CHANGE_ME_STRONG_PASSWORD' | docker secret create grafana_admin_password -
printf '%s' 'CHANGE_ME_SSF_DB'        | docker secret create ssf_db_password -
```

---

## 6) Deploy

```bash
cd docker-swarm
docker stack deploy -c stack.yml lgtm
```

---

## 7) Access Grafana

Open:

http://localhost:3000

Login:
- user: admin
- password: value of `grafana_admin_password` secret

---

## 8) Application Labels

### Metrics

```yaml
deploy:
  labels:
    metrics: prometheus
    metrics_port: "8080"
    metrics_path: /actuator/prometheus
```

### Logs

```yaml
deploy:
  labels:
    logging: promtail
```

---

## 9) Update / Rollback

```bash
docker stack deploy -c stack.yml lgtm
docker service rollback lgtm_grafana
```

---

## 10) Remove Stack

```bash
docker stack rm lgtm
sudo rm -rf ./observability /dashboards
```

---

## Notes

- Placement constraints are kept for future multi-node expansion
- docker-socket-proxy is mandatory
- This setup is production-safe for single-node Swarm
