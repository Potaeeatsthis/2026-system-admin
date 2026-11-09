# Graph Report - 2026-system-admin  (2026-11-10)

## Corpus Check
- 8 files · ~6,198 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 71 nodes · 139 edges · 12 communities
- Extraction: 96% EXTRACTED · 4% INFERRED · 0% AMBIGUOUS · INFERRED: 5 edges (avg confidence: 0.93)
- Token cost: unavailable. Host-agent usage is not exposed; stored zero counters are placeholders, not a claim of free extraction.

## Community Hubs (Navigation)
- Network and Connectivity
- Dashboard Controls and Panels
- Central Monitoring Services
- Web Server Services
- Azure Costs and Quotas
- Administration and Safe Changes
- Log Collection and Labels
- Service Checks and Pending Work
- Assignment Requirements
- Private Network Design
- Grafana Access and Data Sources
- Project Documentation

## God Nodes (most connected - your core abstractions)
1. `Monitoring services and dashboard guide` - 22 edges
2. `Network address plan and connectivity` - 20 edges
3. `Monitor VM: B2als_v2 in Indonesia Central` - 11 edges
4. `Web Server Logs dashboard JSON` - 10 edges
5. `Monitor — private IP 172.17.0.4` - 9 edges
6. `Three web VMs: B2ats_v2 in Malaysia West` - 8 edges
7. `vnet-malaysiawest-1` - 6 edges
8. `Loki log storage and label API` - 6 edges
9. `Grafana Explore displays web-server-01 and 02 logs` - 6 edges
10. `Prometheus collection — all four targets up` - 6 edges

## Surprising Connections (you probably didn't know these)
- `Monitor VM: B2als_v2 in Indonesia Central` --conceptually_related_to--> `Network monitoring system`  [INFERRED]
  docs/azure-handbook.md → README.md
- `Retain VMs and readdress Monitor before peering` --references--> `Non-overlapping network ranges`  [EXTRACTED]
  graphify-out/memory/query_20261109_164812_dd2f871e_can_we_fix_overlapping_vm_networks_without_reinsta.md → docs/azure-handbook.md
- `Retain VMs and readdress Monitor before peering` --references--> `Monitor subnet migration without reinstall`  [EXTRACTED]
  graphify-out/memory/query_20261109_164812_dd2f871e_can_we_fix_overlapping_vm_networks_without_reinsta.md → docs/network-status.md
- `Network monitoring system` --references--> `Network address plan and connectivity`  [EXTRACTED]
  README.md → docs/network-status.md
- `Alloy to Loki log delivery for web-server-01 and 02` --semantically_similar_to--> `Loki log storage and label API`  [INFERRED] [semantically similar]
  docs/network-status.md → docs/monitoring-setup.md

## Communities (12 total, 0 thin omitted)

### Community 0 - "Network and Connectivity"
Cohesion: 0.29
Nodes (17): Alloy to Loki log delivery for web-server-01 and 02, indonesia-to-malaysia — Connected and Fully Synchronized, vnet-indonesiacentral-1, malaysia-to-indonesia — reverse-link check pending, vnet-malaysiawest-1, Monitor — private IP 172.17.0.4, monitor567 NIC with monitor-nsg, Monitor administrator SSH access (+9 more)

### Community 1 - "Dashboard Controls and Panels"
Cohesion: 0.46
Nodes (8): Promtail Kubernetes dashboard incompatibility, NGINX access and error-log entry counts, Single log-type dropdown for one log viewer, Wrapped log viewer — newest first, 1,000-line limit, Monitoring services and dashboard guide, Server dropdown filters every panel by host, Five-minute NGINX access-log rate by server, Web Server Logs dashboard JSON

### Community 2 - "Central Monitoring Services"
Cohesion: 0.40
Nodes (5): Alertmanager role — setup pending, Grafana dashboard role, Loki log storage role, Monitor VM: B2als_v2 in Indonesia Central, Prometheus monitoring role

### Community 3 - "Web Server Services"
Cohesion: 0.40
Nodes (5): Alloy log collection role, NGINX required on every VM, Node Exporter host metrics role, Private IPs and cross-network DNS, Three web VMs: B2ats_v2 in Malaysia West

### Community 4 - "Azure Costs and Quotas"
Cohesion: 0.40
Nodes (5): Compute cost and student credit, Regional and VM-family CPU quotas, Stop and deallocate after work, Indonesia Central, Malaysia West

### Community 5 - "Administration and Safe Changes"
Cohesion: 0.40
Nodes (5): rg-sysadmin resource group, Safe deletion and leftover cleanup, IP-restricted SSH firewall rules, SSH keys and laptop configuration, Retain VMs and readdress Monitor before peering

### Community 6 - "Log Collection and Labels"
Cohesion: 0.40
Nodes (5): Alloy installed on three web servers, Five log categories: auth, kernel, nginx_access, nginx_error, syslog, Alloy log-file permissions and deployed configuration, LogQL selectors must match configured job labels, Loki log storage and label API

### Community 7 - "Service Checks and Pending Work"
Cohesion: 0.50
Nodes (5): Node Exporter service — active and enabled on web-server-01, Prometheus collection — four targets up every 15 seconds, Pending checks: third-server logs, reboot recovery, alerts and networks, Web-server-03 log delivery — check pending, Node Exporter service and Prometheus collection checks

### Community 8 - "Assignment Requirements"
Cohesion: 0.50
Nodes (5): Assignment requirements, Demonstration and instructor questions, Email alerts and triggered tests, Searchable central logs, Host metrics and dashboard

### Community 9 - "Private Network Design"
Cohesion: 0.83
Nodes (4): Indonesia VNet target range 172.17.0.0/16, Malaysia private network, Non-overlapping network ranges, Private connection between regions

### Community 10 - "Grafana Access and Data Sources"
Cohesion: 0.50
Nodes (4): Grafana Explore displays web-server-01 and 02 logs, Grafana data source URLs on Monitor, Local SSH tunnels for dashboard access, Grafana log display and metrics integration

### Community 11 - "Project Documentation"
Cohesion: 1.00
Nodes (3): Azure operating handbook, Assignment brief and project requirements, Network monitoring system

## Knowledge Gaps
- **7 isolated node(s):** `Demonstration and instructor questions`, `Alertmanager role — setup pending`, `Grafana dashboard role`, `Loki log storage role`, `Prometheus monitoring role` (+2 more)
  These have ≤1 connection - possible missing edges or undocumented components. (Counts symbols only; 8 node(s) total have ≤1 connection when file, concept and rationale nodes are included.)

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Monitoring services and dashboard guide` connect `Dashboard Controls and Panels` to `Network and Connectivity`, `Log Collection and Labels`, `Service Checks and Pending Work`, `Grafana Access and Data Sources`, `Project Documentation`?**
  _High betweenness centrality (0.368) - this node is a cross-community bridge._
- **Why does `Network address plan and connectivity` connect `Network and Connectivity` to `Dashboard Controls and Panels`, `Grafana Access and Data Sources`, `Project Documentation`?**
  _High betweenness centrality (0.340) - this node is a cross-community bridge._
- **Why does `Azure operating handbook` connect `Project Documentation` to `Network and Connectivity`, `Dashboard Controls and Panels`, `Central Monitoring Services`, `Web Server Services`?**
  _High betweenness centrality (0.237) - this node is a cross-community bridge._
- **What connects `Demonstration and instructor questions`, `Alertmanager role — setup pending`, `Grafana dashboard role` to the rest of the system?**
  _7 weakly-connected nodes found - possible documentation gaps or missing edges._
## Build scope

Updated 10 November 2026 (Asia/Bangkok). The graph indexes the project documentation and assignment requirements. The dashboard behavior is described in docs/monitoring-setup.md. Final dashboard import, web-server-03 log delivery, alerts, and reboot recovery remain pending checks.

Raw incremental extraction integrity: no dangling/missing endpoints, self-loops, or collapsed endpoint pairs.
