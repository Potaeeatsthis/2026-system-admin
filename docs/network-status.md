# Network status

Updated 10 November 2026 (Asia/Bangkok). Use the
[Azure handbook](azure-handbook.md) for setup procedures and the
[monitoring guide](monitoring-setup.md) for service checks.

## Address plan

| VM | VNet | Subnet | Private IP |
|---|---|---|---|
| monitor | vnet-indonesiacentral-1 | snet-monitor-new (172.17.0.0/24) | 172.17.0.4 |
| web-server-01 | vnet-malaysiawest-1 | snet-malaysiawest-1 | 172.16.0.4 |
| web-server-02 | vnet-malaysiawest-1 | snet-malaysiawest-1 | 172.16.0.5 |
| web-server-03 | vnet-malaysiawest-1 | snet-malaysiawest-1 | 172.16.0.6 |

Monitor uses NIC `monitor567` and NSG `monitor-nsg`. Its public address is written
as `MONITOR_PUBLIC_IP`; obtain the current value from Azure when connecting.

## Subnets and peering

Monitor moved to `snet-monitor-new` to separate its private addresses from the web
servers. This avoids reinstalling the VMs. The target VNet ranges are
`172.16.0.0/16` for Malaysia and `172.17.0.0/16` for Indonesia.

Before removing Indonesia's original `172.16.0.0/24` subnet, confirm it has no
attached resources. Remove the old overlapping VNet range only after checking
that it is unused. Final address-space cleanup remains a pending check.

| Peering | Status |
|---|---|
| indonesia-to-malaysia | Connected and Fully Synchronized at the last check |
| malaysia-to-indonesia | Reverse-link status needs checking |

Allow access between the VNets. Gateway transit, remote gateways, and forwarded
traffic are not needed for this topology. Link names identify the peering entries;
they do not replace routing and firewall configuration.

## Connectivity

| Connection | Status |
|---|---|
| Administrator → Monitor SSH | Working at the last check |
| Monitor → web-server-01 private TCP 9100 | Metrics request passed |
| Prometheus → web-server-01, 02, and 03 Node Exporter | All targets up, no scrape errors, 15-second interval |
| Prometheus → its own metrics endpoint | Up |
| Web-server-01 and 02 → Loki → Grafana | Logs available in Grafana Explore |
| Web-server-03 → Loki → Grafana | Log delivery needs checking |
| Monitor → web VMs private TCP 22 | Needs a test from inside Monitor |

Successful metrics collection does not establish private SSH access. Test each
port separately. Short hostnames resolved for the successful Prometheus requests;
the DNS or hosts-file configuration still needs documenting. VNet peering does
not automatically provide cross-VNet hostname resolution.

To test private SSH, run inside Monitor:

```bash
hostname
hostname -I
nc -vz -w 5 172.16.0.4 22
nc -vz -w 5 172.16.0.5 22
nc -vz -w 5 172.16.0.6 22
```

Confirm that the machine is Monitor at `172.17.0.4`. A laptop without a VPN or
another private route cannot normally access these Azure private addresses.
If a connection times out, check peering in both directions, effective NSG rules,
the guest firewall, the service listener, VM state, and routes.

## Access rules

- Use private TCP 9100 for Prometheus collection from web servers.
- Use private TCP 3100 for Alloy delivery to Loki on Monitor.
- Restrict public SSH to each administrator's current public IPv4 with `/32`.
- Keep Grafana, Prometheus, Alertmanager, Loki, and exporter ports off the public
  internet; use SSH tunnels for dashboard access.
- Review effective network rules and listening addresses before deployment.
  Current collection success does not prove that public exposure is blocked.

Run `curl -4 https://api.ipify.org` on the administrator's laptop to find its
outbound public IPv4. `hostname -I` shows local interface addresses instead.
Public addresses can change; update narrow SSH rules when needed.
