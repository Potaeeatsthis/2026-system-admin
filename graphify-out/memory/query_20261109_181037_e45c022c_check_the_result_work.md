---
type: "query"
date: "2026-11-09T18:10:37.231712+00:00"
question: "How do we check Node Exporter?"
contributor: "graphify"
outcome: "useful"
source_nodes: ["Node Exporter service", "Prometheus collection"]
---

# Q: How do we check Node Exporter?

## Answer

Check `node_exporter.service`, request `/metrics`, and inspect Prometheus target
health. Node Exporter 1.10.2 uses TCP 9100. On web-server-01, the systemd service
is active and enabled and runs `/usr/local/bin/node_exporter`.

Prometheus collection passed for all three web servers and itself, with a
15-second interval. Service startup and metrics collection are separate checks;
reboot recovery still needs testing. See the
[monitoring guide](../../docs/monitoring-setup.md) for commands.

## Outcome

- Signal: useful

## Source Nodes

- Node Exporter service
- Prometheus collection
