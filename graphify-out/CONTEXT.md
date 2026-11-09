# Project context: Network Monitoring System

Updated 10 November 2026 (Asia/Bangkok).

## Purpose

Collect Linux host metrics and logs centrally, display them in Grafana, and add
email alerts for the required failure conditions. The
[assignment brief](../docs/requirements.pdf) defines the requirements.

## Infrastructure

| VM | Region | Size | Private IP | Service layout |
|---|---|---|---|---|
| monitor | Indonesia Central | B2als_v2, 2 vCPUs / 4 GiB | 172.17.0.4 | Prometheus, Loki, Grafana; Alertmanager and NGINX checks pending |
| web-server-01 | Malaysia West | B2ats_v2, 2 vCPUs / 1 GiB | 172.16.0.4 | NGINX, Node Exporter, Alloy |
| web-server-02 | Malaysia West | B2ats_v2, 2 vCPUs / 1 GiB | 172.16.0.5 | NGINX, Node Exporter, Alloy |
| web-server-03 | Malaysia West | B2ats_v2, 2 vCPUs / 1 GiB | 172.16.0.6 | NGINX, Node Exporter, Alloy |

The two-region layout fits Malaysia West's six-vCPU quota during setup. Check
current regional and VM-family quotas before adding or resizing VMs.

## Service status

- All three web Node Exporters and Prometheus itself passed collection checks.
- Prometheus collection interval is 15 seconds, with a 10-second timeout.
- Node Exporter on web-server-01 runs as the enabled `node_exporter.service`.
- Alloy is installed on all web VMs. Logs from web-server-01 and web-server-02
  are available in Grafana; web-server-03 log delivery needs checking.
- Loki job labels are `auth`, `kernel`, `nginx_access`, `nginx_error`, and `syslog`.
- The importable dashboard has one log viewer, one log-type dropdown, a server
  filter, counts, and a traffic graph. Its final import still needs testing.

See [monitoring setup](../docs/monitoring-setup.md) and the
[dashboard JSON](../dashboards/web-server-logs.json).

## Networking and security

- Resource group: `rg-sysadmin`.
- Malaysia VNet: `vnet-malaysiawest-1`, target range `172.16.0.0/16`.
- Indonesia VNet: `vnet-indonesiacentral-1`, target range `172.17.0.0/16`.
- Monitor uses `snet-monitor-new`, `172.17.0.0/24`, after subnet migration.
- Indonesia-to-Malaysia peering passed its status check. Reverse-link status and
  removal of the old overlapping Indonesia address range still need checking.
- Monitor SSH works. Private SSH from Monitor to each web VM needs a separate test.
- Use private monitoring traffic, SSH-key authentication, narrow public SSH rules,
  and laptop SSH tunnels. Effective firewall restrictions still need reviewing.
- Keep public addresses as placeholders and credentials outside Git.

See [network status](../docs/network-status.md) and the
[Azure handbook](../docs/azure-handbook.md).

## Next steps

1. Verify log delivery from web-server-03 and import the final dashboard.
2. Test service restart and reboot behavior on every VM.
3. Complete uptime, service status, CPU, memory, network, disk I/O, and free-space
   monitoring against the assignment requirements.
4. Configure and test email alerts, including firing and resolved notifications.
5. Verify NGINX on Monitor, final VNet ranges, reverse peering, and access rules.

## Operating costs

Stop VMs in Azure and confirm **Stopped (deallocated)** after work. Closing SSH
leaves VMs running. Deallocation stops compute charges, but disks and public IPs
can still cost money, and deallocated VMs still count against vCPU quotas.
Check current prices and student credit before extending the deployment.
