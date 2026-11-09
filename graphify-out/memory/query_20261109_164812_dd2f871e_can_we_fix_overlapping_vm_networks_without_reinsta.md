---
type: "query"
date: "2026-11-09T16:48:12.546358+00:00"
question: "Can we fix overlapping VM networks without reinstalling the VMs?"
contributor: "graphify"
outcome: "useful"
source_nodes: ["Non-overlapping network ranges", "Private connection between regions"]
---

# Q: Can we fix overlapping VM networks without reinstalling the VMs?

## Answer

Use non-overlapping address ranges for the two VNets: `172.16.0.0/16` for
Malaysia and `172.17.0.0/16` for Indonesia. Move Monitor's NIC to
`snet-monitor-new` (`172.17.0.0/24`) within the existing Indonesia VNet instead of
reinstalling the VMs. Monitor now uses `172.17.0.4`.

Remove old subnets and overlapping ranges only after confirming that they are
unused. Check peering in both directions and test the required private ports.
Keep the web VNet and VM disks; deleting the resource group is not a network fix.
See [network status](../../docs/network-status.md) for pending network checks.

## Outcome

- Signal: useful

## Source Nodes

- Non-overlapping network ranges
- Private connection between regions
