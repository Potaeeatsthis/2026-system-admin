# Network Monitoring System

A centralized view of the health, performance, and logs of Linux webserver VMs.
Course project for **01276521 Practice of System Administration**, Computer
Engineering International Program, King Mongkut's Institute of Technology
Ladkrabang (KMITL).

> **Source of truth:** [`docs/requirements.pdf`](docs/requirements.pdf). If this
> README conflicts with the brief, the brief wins.

## Purpose

Give operators one place to observe host and service health, traffic, resource
usage, and logs, instead of inspecting each VM separately. The system is planned
to collect from multiple webserver hosts and surface the results in dashboards
and email alerts.

## Planned capabilities

- **Log collection:** central, searchable access to system logs (authentication
  and kernel), web server access logs, and web server error logs.
- **Metrics and dashboards:** uptime, service status, CPU and memory
  utilization, inbound/outbound network traffic, disk read/write throughput,
  and free disk space.
- **Email alerts:** notifications for conditions such as repeated unauthorized
  sudo attempts, error or critical web server log entries, a stopped web server,
  high CPU or memory utilization, elevated inbound traffic, high disk write
  throughput, and low free disk space.

## Proposed architecture

The planned topology is four Linux VMs: at least three webserver hosts, plus
central monitoring and log services that collect from them.
The assignment requires a web server on every VM, including any central host.

This is a proposed layout for planning and is neither implemented nor a mandated
design. Monitoring and log software must be open-source, self-hosted, and
installed manually. Hosting the VMs on a public cloud provider is allowed, but
fully managed monitoring or log services are not.

## Status

This repository currently contains project documentation only. Tool choices,
deployment configuration, and dashboards are not yet documented here, and no
monitoring stack is implemented in this repository.

## Repository contents

```
.
├── README.md
├── .gitignore
└── docs/
    └── requirements.pdf   # assignment brief (source of truth)
```

## Security

Do not commit credentials, private keys, or cloud account secrets. Keep local
secrets out of version control; `.gitignore` excludes `keys/` and local tool and
cloud configuration directories.
