# Operating notes

## Network changes

Use non-overlapping VNet address ranges. Monitor can move to another subnet in
its existing VNet without reinstalling the VM. Check attached resources before
removing old subnets or address ranges. Test peering and the required private
ports after network changes.

## Service checks

Check service state, endpoint access, and successful collection separately.
A running process does not prove that Prometheus or Loki receives data.
An enabled systemd unit still needs a reboot test.

## Log queries

Use the configured job labels: `auth`, `kernel`, `nginx_access`, `nginx_error`,
and `syslog`. A query for `job="nginx"` does not match these categories.
An empty error-log panel does not prove a server is healthy.

## Related guides

- [Network status](../../docs/network-status.md)
- [Monitoring setup](../../docs/monitoring-setup.md)
