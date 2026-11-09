# Network Monitoring System

Central monitoring for Linux web servers: host metrics, searchable logs, and
Grafana dashboards.

Course project for **01276521 Practice of System Administration**, Computer
Engineering International Program, King Mongkut's Institute of Technology
Ladkrabang (KMITL).

The [assignment brief](docs/requirements.pdf) defines the requirements and marking
criteria. It takes priority if another document disagrees with it.

## Architecture

The system uses four Ubuntu VMs on Azure: three web servers in Malaysia West and
one Monitor VM in Indonesia Central. Private VNet peering connects the regions.

| Data | Collection | Storage | Display |
|---|---|---|---|
| Host metrics | Node Exporter on web servers | Prometheus on Monitor | Grafana |
| Web and system logs | Alloy on web servers | Loki on Monitor | Grafana |

Monitoring software is open-source, self-hosted, and installed on the VMs.
The assignment also requires NGINX or another web server on every VM, including
Monitor. Fully managed monitoring services are outside the project scope.

## Current status

Updated 10 November 2026 (Asia/Bangkok).

- Prometheus collects from all three web servers and itself every 15 seconds.
  All four targets passed the latest collection check.
- Node Exporter on web-server-01 runs as an enabled systemd service using
  `/usr/local/bin/node_exporter`.
- Alloy is installed on all three web servers. Log delivery to Grafana has been
  checked for web-server-01 and web-server-02; web-server-03 still needs checking.
- Loki has five log categories: authentication, kernel, NGINX access, NGINX
  errors, and system logs.
- The **Web Server Logs** dashboard includes server and log-type filters, access
  and error entry counts, a traffic graph, and one log viewer. Its JSON passes
  local checks; import and final filter behavior still need testing in Grafana.

Pending work: automatic recovery after reboot, email alerts, full metric and
alert coverage, and NGINX on Monitor. See the
[monitoring guide](docs/monitoring-setup.md) for checks and next steps.

## Dashboard import

1. Open Grafana → Dashboards → New → Import.
2. Upload [dashboards/web-server-logs.json](dashboards/web-server-logs.json).
3. Select the Loki data source and click Import.

The Server filter applies to every panel. Log type changes only the log viewer.
NGINX access is selected by default; All displays all five categories.

## Project files

| File | Purpose |
|---|---|
| [Azure handbook](docs/azure-handbook.md) | VM setup, quotas, private networking, SSH, and costs |
| [Network status](docs/network-status.md) | Address table, peering, access, and connectivity checks |
| [Monitoring guide](docs/monitoring-setup.md) | Service checks, log queries, and dashboard use |
| [Dashboard JSON](dashboards/web-server-logs.json) | Importable Grafana dashboard |
| [Graph report](graphify-out/GRAPH_REPORT.md) | Project relationships and navigation |
| [Interactive graph](graphify-out/graph.html) | Browse the project knowledge graph |

Services run on Azure; the repository does not contain their complete deployed
configurations.

## Security

Keep credentials, private keys, and cloud secrets out of Git. Public server
addresses use placeholders. Access dashboards through SSH tunnels and restrict
service traffic to the intended private networks. The `.gitignore` excludes local
keys, environment secrets, and generated caches; it cannot protect secrets that
were already committed.
