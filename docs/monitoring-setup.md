# Monitoring setup and dashboard

Updated 10 November 2026 (Asia/Bangkok). This guide describes the service layout,
current checks, and dashboard setup. Run the checks again after configuration
changes or restarts.

## How the services connect

- Metrics: Node Exporter on each web VM → Prometheus on Monitor → Grafana.
- Logs: Alloy on each web VM → Loki on Monitor → Grafana.
- Node Exporter exposes measurements; Prometheus requests and stores them.
- Alloy reads log files and sends entries; Loki stores them; Grafana queries Loki.

| Service | Host | Port | Status |
|---|---|---|---|
| Node Exporter 1.10.2 | Web VMs | 9100 | All three targets reported up; web-server-01 service active and enabled |
| Prometheus | Monitor | 9090 | Targets API returned success, all four targets up |
| Alloy | Web VMs | Local UI 12345 | Installed on all three; web-server-01 reads five log types |
| Loki | Monitor | 3100 | Job-label API succeeded; Grafana displayed stored logs |
| Grafana | Monitor | 3000 | Running; logs available in Explore |
| Alertmanager | Monitor, intended | 9093 | Installation and alert delivery unverified |

Monitor's last documented private IP is `172.17.0.4`. Web VM private IPs are
`172.16.0.4`, `172.16.0.5`, and `172.16.0.6`. Recheck addresses before changing
configuration. See [network status](network-status.md) for limitations.

## Metrics and service checks

Prometheus lists `web-server-01:9100`, `web-server-02:9100`,
`web-server-03:9100`, and `localhost:9090` as `up`, with empty `lastError` values,
a 15-second interval, and a 10-second timeout. Collection passed at the last check. Repeat the check after changes.

On Monitor, inspect current targets:

```bash
curl -fsS http://localhost:9090/api/v1/targets | python3 -c 'import sys,json; [print(t["labels"]["instance"], t["health"]) for t in json.load(sys.stdin)["data"]["activeTargets"]]'
```

On a web VM, check the manually created service:

```bash
systemctl is-active node_exporter
systemctl is-enabled node_exporter
sudo systemctl status node_exporter --no-pager
curl -fsS http://localhost:9100/metrics -o /dev/null
```

The manual unit is `node_exporter.service`, not `prometheus-node-exporter`.
On web-server-01 it runs `/usr/local/bin/node_exporter` and was active and enabled.
Enabled means configured to start at boot; an actual reboot test remains undone.
The standalone process started with `./node_exporter` is not the systemd service.
Stop a manual foreground instance before starting a service on the same port.

The official example uses a different binary path and a companion socket unit;
do not mix its `--web.systemd-socket` command with our standalone service.

`curl ... | head` can show error 23 because `head` closes the pipe early. To
inspect a sample without that error:

```bash
curl -fsS http://localhost:9100/metrics -o /tmp/node-metrics.txt
head /tmp/node-metrics.txt
```

Delete the downloaded Node Exporter archive and extracted folder only after
checking that the service uses the copied binary outside that folder. Do not
assume web-server-01's service configuration proves another VM's configuration.

## Log collection and checks

Alloy on web-server-01 watches these files:

| Job label returned by Loki | Expected log file on Ubuntu |
|---|---|
| `nginx_access` | `/var/log/nginx/access.log` |
| `nginx_error` | `/var/log/nginx/error.log` |
| `auth` | `/var/log/auth.log` |
| `kernel` | `/var/log/kern.log` |
| `syslog` | `/var/log/syslog` |

Check each file-to-label mapping in `/etc/alloy/config.alloy` before editing.
Standard file paths can vary. Grant read access to the Alloy service user without
making sensitive logs public. The complete deployed configuration is not stored
in this repository.

Loki returned:

```json
{"status":"success","data":["auth","kernel","nginx_access","nginx_error","syslog"]}
```

Grafana Explore displays logs from web-server-01 and web-server-02. Alloy is
installed on all three web VMs; log delivery from web-server-03 still needs
checking. A running Alloy process or an open file reader alone does not prove
successful delivery to Loki.

On Monitor:

```bash
curl -fsS http://localhost:3100/ready
curl -fsS http://localhost:3100/loki/api/v1/label/job/values
curl -fsS http://localhost:3100/loki/api/v1/label/host/values
```

On a web VM:

```bash
systemctl is-active alloy
sudo alloy validate /etc/alloy/config.alloy
sudo journalctl -u alloy --since "10 minutes ago" --no-pager
sudo tail -n 5 /var/log/nginx/access.log
```

Generate a fresh access entry if needed:

```bash
curl -s -o /dev/null http://localhost/alloy-log-test
```

A 404 response normally still creates an access-log entry. In Grafana Explore,
select Loki and Last 1 hour, then use:

```logql
{job=~"nginx_access|nginx_error"}
```

To verify the third server, assuming its host label matches the VM name:

```logql
{job=~"auth|kernel|nginx_access|nginx_error|syslog", host="web-server-03"}
```

The query `{job="nginx"}` does not match the configured labels. `UNK` or unknown
severity means Grafana did not identify a log level, not that collection failed.
Bangkok display time is UTC+7; a raw UTC timestamp seven hours earlier can be the
same event. Compare the selected time range before diagnosing missing entries.

## Import the Web Server Logs dashboard

Import [dashboards/web-server-logs.json](../dashboards/web-server-logs.json).
This is the final requested version: one log viewer and one single-selection
Log type dropdown, with an All option. Server selection allows multiple servers.

1. Open Grafana → Dashboards → New → Import.
2. Upload the JSON file.
3. Select the working Loki data source when prompted.
4. Import and save. If a dashboard with this title already exists, choose a
   different name or explicitly select the dashboard you intend to replace.

| Control or panel | Behavior |
|---|---|
| Server | Filters every data panel using the `host` label |
| Log type | Filters only the single log viewer; defaults to NGINX access |
| Access log entries | Count over the selected dashboard time range |
| Error log entries | Count of NGINX error-log entries, not failed HTTP requests |
| NGINX traffic by server | Five-minute average access-log entries per second |
| Log viewer | Newest first, wrapped lines, up to 1,000 matching entries |

The dashboard uses a browser timezone, a default Last 1 hour range, and 30-second
refresh. Click a log line to inspect labels. Narrow the time range or filters
when the 1,000-line display limit hides older entries. Empty results are not proof
of a healthy server: they can mean no matching activity or failed collection.

The dashboard passes local checks for JSON syntax, data source references,
dropdown selection, and panel layout. Final import and filter behavior need
testing in Grafana. Use the repository JSON as the canonical dashboard file.

The previously imported Loki stack monitoring (Promtail, Loki) dashboard expects
Kubernetes metrics, Promtail, and `app="loki"`/`app="promtail"` log labels. It does
not match our VM and Alloy setup. Do not add Kubernetes or Promtail just to fill
those panels. Loki internal metrics require a separate Prometheus scrape job;
this has not been verified.

## Private access and remaining work

Use laptop SSH tunnels rather than exposing monitoring ports publicly:

```bash
ssh -N -o ExitOnForwardFailure=yes \
  -L 127.0.0.1:9090:localhost:9090 \
  -L 127.0.0.1:3000:localhost:3000 \
  -L 127.0.0.1:9093:localhost:9093 monitor
```

Open `http://localhost:3000` on the laptop for Grafana. With services installed
directly on Monitor, Grafana's data source URLs are `http://localhost:9090` for
Prometheus and `http://localhost:3100` for Loki. Here localhost means Monitor
because Grafana's server makes the requests. Containers may need different URLs.

A running tunnel does not prove that the destination service is running. Keep
TCP 3100 restricted to intended log senders and TCP 9100 to the monitoring source;
actual effective firewall rules and listener restrictions remain unverified.

## Remaining checks

- Logs from web-server-03 and final dashboard import/filters.
- Automatic restart and boot behavior of services on each VM.
- Alertmanager and email alert delivery, including controlled trigger tests.
- Complete assignment metric/alert coverage and NGINX on Monitor.
- Current effective network rules, reverse peering status, and private SSH tests.
