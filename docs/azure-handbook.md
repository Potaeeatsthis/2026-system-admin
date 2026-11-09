# Azure handbook for our monitoring project

This guide covers VM creation, private networking, SSH access, quotas, costs, and safe resource cleanup for the monitoring system. Portal labels may change slightly.

**Updated:** 10 November 2026 (Asia/Bangkok). Check current Azure resource settings, prices, and quotas before making changes. See [monitoring setup](monitoring-setup.md) for service checks and [network status](network-status.md) for addresses and connectivity.

## 1. Our four VMs

| VM | Region | Size | CPU / RAM | Purpose |
|---|---|---|---|---|
| `monitor` | Indonesia Central | `B2als_v2` | 2 vCPUs / 4 GiB | Prometheus, Alertmanager, Loki, Grafana, and NGINX |
| `web-server-01` | Malaysia West | `B2ats_v2` | 2 vCPUs / 1 GiB | NGINX, node_exporter, and Alloy |
| `web-server-02` | Malaysia West | `B2ats_v2` | 2 vCPUs / 1 GiB | Same as web-server-01 |
| `web-server-03` | Malaysia West | `B2ats_v2` | 2 vCPUs / 1 GiB | Same as web-server-01 |

We split the regions because Malaysia West's last confirmed regional limit was **6 vCPUs**. The three web VMs use all six. The monitor uses Indonesia Central's separate quota. The quota increase request for Malaysia West was unsuccessful.

The assignment allows cloud VMs, but the monitoring tools must be self-hosted on Linux. Every VM needs a web server, including the monitor. See [the assignment](requirements.pdf).

## 2. What the Azure terms mean

| Term | Meaning | Why it matters here |
|---|---|---|
| Subscription | The account boundary for billing, access, and quotas | Select Azure for Students |
| Student credit | Money provided by the student offer | Resources use this balance; check its remaining amount and expiry |
| Resource group | A container for related resources | We use `rg-sysadmin` to organise the project |
| Region | The Azure location hosting a resource | It affects policy, prices, available sizes, and quotas |
| VM | A virtual computer | Runs Ubuntu and our services |
| vCPU | A virtual processor available to the VM | Counts against CPU quotas |
| RAM | Working memory for running programs | Too little can cause slowdowns or killed processes |
| OS disk | Persistent storage containing Ubuntu and files | It can keep billing after the VM stops |
| VNet | A private network in Azure | Lets VMs communicate through private IPs |
| Subnet | A smaller address range inside a VNet | The VM's network interface connects to it |
| NIC | Network interface attached to a VM | Holds private IP settings and can have an NSG |
| Public IP | An internet-facing address | Used for SSH from your laptop; separately billed |
| Private IP | An internal address | Used for monitoring and log traffic |
| NSG | Network Security Group: Azure traffic-filtering rules | Limits which connections can reach our VMs |
| VNet peering | A private connection between VNets | Connects the monitor to the web VMs across regions |

The resource group's location stores its metadata. It does **not** force every resource into that region. A group whose location is Southeast Asia can contain VMs in Malaysia West and Indonesia Central.

## 3. Before creating a VM

### Check allowed regions

Open **Policy → Authoring → Assignments**, with scope set to **Azure for Students**. Open **Allowed resource deployment regions → Parameters**.

The subscription allowed Malaysia West, Indonesia Central, and East Asia during setup. Confirm the current list rather than guessing. A region shown in the VM dropdown or quota table can still be blocked by policy.

### Check CPU quota

Open **Quotas → Compute**. Select the subscription and intended region. Check:

1. **Total Regional vCPUs:** the total permitted in that region.
2. **The selected VM family's vCPU quota:** a separate limit for that family.

Both limits must have enough space. Do not assume the old BS-family quota applies to newer B-series v2 sizes.

An increase request can fail even when the portal says **Adjustable: Yes**. Increasing quota does not itself cost money; running resources does.

### Register a provider if Azure asks

Open **Subscriptions → Azure for Students → Settings → Resource providers**. Search `Microsoft.Compute`, select **Register**, then refresh until registered.

A provider enables an Azure service type. Registration itself is free. You do not need to register every provider whose name contains Compute.

## 4. Create or recreate a VM

Open **Virtual machines → Create → Azure virtual machine**.

| Setting | Our choice and reason |
|---|---|
| Resource group | `rg-sysadmin`, to keep project resources together |
| Name, region, size | Use the table in section 1 |
| Availability | No infrastructure redundancy required; sufficient for this lab |
| Image | Ubuntu Server 24.04 LTS, x64 |
| Security | Trusted Launch, Secure Boot, and vTPM enabled |
| Authentication | SSH public key |
| Username | `azureuser`; a normal login account with administrator access through `sudo` |
| OS disk | Standard SSD, default image capacity |
| Spot discount | Off; Azure can evict Spot VMs and interrupt the demo |
| Backup / Site Recovery | Off for this temporary lab unless you choose to protect important data |
| Boot diagnostics | On, to help investigate boot problems |
| Delete OS disk with VM | On only if you accept losing that disk when deleting the VM |
| Delete public IP and NIC with VM | On for these dedicated lab resources, to reduce leftover resources |

In **Networking**, select the correct regional VNet and subnet. The three web VMs reuse Malaysia's network and `web-malaysia-nsg`. The monitor uses a separate Indonesia network and regional NSG.

You can reuse a stored public key or generate a new pair. RSA and Ed25519 are both valid when supported by your SSH client. If generating a key, download the private key and record which VM uses it.

The **Public inbound ports** summary is not a complete firewall check. Inspect the actual NSG rules, especially when using Advanced networking.

Select **Review + create**, read validation errors and the price, then create. Wait for deployment completion before testing SSH.

### Names and regions after creation

Azure VM resource names cannot be renamed directly. A Linux hostname or SSH shortcut can be changed independently, but neither renames the Azure resource.

For our new, mostly empty monitor, recreating it in another region was simpler than migrating it. Copy any needed files first. You cannot change its region using a simple settings dropdown.

## 5. Configure the two private networks

Our intended ranges are:

| Network | VNet range | Subnet range |
|---|---|---|
| `vnet-malaysiawest-1` | `172.16.0.0/16` | `172.16.0.0/24` |
| `vnet-indonesiacentral-1` | `172.17.0.0/16` | `172.17.0.0/24` |

The table defines the target address plan. Check both VNets before changing subnets; removal of Indonesia's old overlapping range still needs checking.

`/16` and `/24` describe the size of an IP range. The subnet must fit inside its VNet. The two VNets must not share any addresses, or Azure cannot peer them.

To inspect an existing network, open **Virtual networks → network name → Settings → Address space**. Open **Subnets** to check the subnet range.

While creating a VM, use **Networking → Edit virtual network** and **Edit subnet**. Set both ranges together before deployment. Changing populated networks later needs more planning.

### Connect the regions using peering

After both VNets exist:

1. Open Malaysia's VNet → **Settings → Peerings → Add**.
2. Choose Indonesia's VNet as the remote network.
3. Create links in both directions, using names such as `malaysia-to-indonesia` and `indonesia-to-malaysia`.
4. Allow access between the VNets. Gateway transit, remote gateways, and forwarded traffic are not needed for this simple design.
5. Check that both links show **Connected**.

Portal wording varies; some forms create both links together. Global peering has traffic charges. It connects private networks without requiring a public monitoring endpoint.

**Peering does not automatically make short VM names resolve across VNets.** Use private IPs in the monitoring configuration, or configure DNS deliberately. The original single-VNet assumption that `monitor` and every web hostname resolve everywhere no longer applies.

After services are installed, test from the monitor using a web VM's private IP:

```bash
curl --connect-timeout 5 http://WEB_PRIVATE_IP/
curl --connect-timeout 5 http://WEB_PRIVATE_IP:9100/metrics
```

The second command needs node_exporter running. A timeout can mean a network or firewall issue; connection refused often means no service is listening. NSGs, Ubuntu firewall rules, and service listening addresses must all permit the connection.

Keep Grafana, Prometheus, Alertmanager, Loki, and exporter ports off the public internet. Access dashboards through SSH tunnels.

## 6. Configure the NSG safely

Open **Network security groups → NSG name → Settings → Inbound security rules → Add**.

Use this rule for laptop SSH:

| Field | Value |
|---|---|
| Source | IP Addresses |
| Source IP / CIDR | Your current public IPv4 followed by `/32` |
| Source ports | `*` |
| Destination | Any |
| Destination port | `22` |
| Protocol | TCP |
| Action | Allow |
| Priority | `1010`, or another unused value from 100 to 4096 |
| Name | `Allow-SSH-MyIP` |

`/32` means one IPv4 address. Source ports stay `*` because your laptop chooses a temporary source port. Destination port 22 identifies SSH; 8080 is not our SSH port.

Priorities control checking order: lower numbers are checked first, and the first matching rule decides the result. Priorities must be unique within the same direction of one NSG. Inbound and outbound rules can use the same number.

Remove unnecessary custom Allow rules with source **Any**, **Internet**, or `0.0.0.0/0` if their port includes 22 and their protocol includes TCP. An IP-only rule does not restrict a separate rule that already allows everyone.

Keep the default rules, including `AllowVnetInBound` and `DenyAllInBound`. Default VNet rules support private traffic, including appropriately peered networks; inspect effective rules if traffic fails. An NSG attached to both a subnet and a NIC can create additional restrictions.

To find your laptop's public IPv4:

```bash
curl -4 https://api.ipify.org
```

`hostname -I` shows local interface addresses. It does not reliably show the public IPv4 Azure sees. Update the SSH rule when your public IP changes. Give teammates separate narrow rules for their own IPs if needed.

If the VM form repeatedly loses the source IP, create and save the NSG separately, verify the Source column, then select that existing NSG in **Networking → Advanced**.

## 7. Connect with SSH

Find the VM's **Public IP address** on its Overview page. On your laptop, from the repository directory:

```bash
chmod 400 keys/monitor_key.pem
ssh -o IdentitiesOnly=yes -i keys/monitor_key.pem azureuser@MONITOR_PUBLIC_IP
```

Replace the placeholders and key filename. `chmod 400` allows only your account to read the key. `IdentitiesOnly=yes` prevents SSH from trying many unrelated agent keys.

For the first connection, compare the host fingerprint with the VM's fingerprint through a trusted Azure channel. With **VM → Run command → RunShellScript**, you can display it using:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

The Azure VM agent must be working for Run command. Do not accept an unexpected changed fingerprint without checking why it changed.

### Save a connection shortcut

Edit your laptop's `~/.ssh/config`, keeping any existing entries:

```sshconfig
Host monitor
    HostName MONITOR_PUBLIC_IP
    User azureuser
    IdentityFile /absolute/path/to/keys/monitor_key.pem
    IdentitiesOnly yes
```

Replace the placeholders with real values, then run:

```bash
chmod 600 ~/.ssh/config
ssh monitor
```

Add similar blocks for `web-server-01`, `web-server-02`, and `web-server-03`, with their own public IPs and matching keys. To use `ssh azureuser@PUBLIC_IP` directly, use that numeric IP as the `Host` entry instead of an alias.

After connecting, run `hostname`, `whoami`, and `sudo -l` to check the machine, account, and administrator permissions. `exit` closes SSH but leaves the VM running.

Keep private keys out of Git. This repository excludes `keys/`; that does not protect keys already tracked or previously committed. Never include private key contents in documentation or shared files.

## 8. Costs, stopping, and student credit

Reference compute prices for cost planning:

| VM | Hourly compute | About 24 hours | About 30 days |
|---|---:|---:|---:|
| Monitor | $0.0428 | $1.03 | $30.82 |
| Each web VM | $0.0107 | $0.26 | $7.70 |
| All four together | $0.0749 | $1.80 | $53.93 |

These are calculations using 720 hours for 30 days. Azure's monthly display may use 730 hours, so it can show a different estimate. Disks, public IPs, internet egress, and peering traffic are extra. Check current prices rather than treating these as a bill guarantee.

Azure for Students normally includes $100 credit for 12 months, subject to offer eligibility and terms. Check **Education → Overview** for your actual remaining balance and expiry. Use **Cost Management → Cost analysis** for spending; usage information can be delayed.

### Stop after working

1. Open **Virtual machines** and select the project VMs.
2. Click **Stop** in Azure.
3. Wait until each shows **Stopped (deallocated)**.

Deallocation stops compute charges, but disks and public IPs may keep billing. Closing SSH or running `sudo shutdown` alone is not enough to guarantee deallocation.

If supported, enable **VM → Operations → Auto-shutdown**, selecting Bangkok's UTC+7 timezone. If unavailable, use a personal reminder and stop the VMs manually.

Deallocated VMs still count against Azure vCPU quotas. Do not expect stopping the old monitor to free quota for an extra VM.

## 9. Delete resources without losing the wrong things

**Deletion is not a substitute for stopping.** Use Stop when you want to keep your work. Before deletion, copy needed files and configurations. A snapshot can preserve a disk, but it also costs money.

### Delete an old VM

1. Check its name, region, and public IP so you know which VM it is.
2. Confirm any replacement works and needed files are copied.
3. Open **Virtual machines → old VM → Delete**.
4. Read which disks, NICs, and public IPs Azure will also delete.
5. Keep shared resources; confirm deletion only for the intended resources.
6. Refresh the resource group and quotas afterwards.

Deleting a VM can free its CPU allocation. Deleting an NSG, NIC, or public IP does not free VM CPU quota.

### Remove leftovers

| Resource | Check before deletion | Action |
|---|---|---|
| NIC | Overview shows **Attached to: -** and you no longer need it | Delete the unused NIC |
| Public IP | No association remains, and it is no longer needed | Delete it; future public IPs may differ |
| Disk | Not attached, no needed data, no required recovery copy | Delete only after confirming data loss is acceptable |
| NSG | Not needed by any NIC or subnet | Detach associations, then delete |
| VNet | No VMs, NICs, peering, or planned hosts need it | Delete only when the whole network is no longer needed |
| SSH public-key resource | No longer needed for future deployments | Optional cleanup; protect and manage the laptop's private key separately |

For our project, **keep Malaysia's VNet and subnet while building or using the web VMs**.

### Delete an NSG

Open the NSG's **Network interfaces** and **Subnets** sections. If it is in use, attach a replacement NSG where protection is still needed before removing the old association. Change the NIC's **Network security group** setting or the subnet's NSG association, save, then delete the unused group.

Deleting one rule is different from deleting its entire NSG. To remove a rule, use **Inbound security rules → rule → Delete**. Do not use the NSG Overview's Delete button for that task.

### Delete the whole project

Deleting `rg-sysadmin` deletes the resources inside it, including VM disks selected through that operation. Use **Resource groups → rg-sysadmin → Delete resource group** only when the project is finished and everything needed is backed up. Check for project resources in other groups too.

## 10. Common errors and the next check

| Error or symptom | Meaning / next action |
|---|---|
| `RequestDisallowedByAzure` with allowed-region text | Check the policy's actual allowed locations; changing username or priority will not fix it |
| `NotAvailableForSubscription` | This size cannot deploy with the current subscription/location settings; inspect deployable options and quotas |
| Insufficient regional quota | Add the requested vCPUs to current usage and compare with the regional limit |
| Duplicate priority | Reuse the existing suitable rule or choose another unused inbound priority |
| `SecurityRuleParametersMissing` mentioning source | Enter and save Source = IP Addresses and the source IP/CIDR; verify the saved table |
| Cannot delete NSG because it is in use | Inspect and remove its NIC/subnet associations safely |
| `UNPROTECTED PRIVATE KEY FILE` | Run `chmod 400` on the correct private key |
| Too many authentication failures | Set `IdentitiesOnly yes` and specify the matching key |
| `Permission denied (publickey)` | Check username, key, key permissions, and which public key was installed on that VM |
| SSH timeout | Check Running status, current public IP, your source IP, NSG attachment/rules, and Ubuntu firewall |
| VM agent not ready | Refresh after initial boot; if persistent, inspect boot diagnostics and guest agent status; SSH may still work |
| Peering connected but names fail | Test private IPs first; cross-VNet hostname resolution needs DNS configuration |

## 11. End-of-session checklist

- Save configuration notes and private IPs, without recording credentials in Git.
- Confirm private monitoring ports are not publicly exposed.
- Stop all four VMs in Azure and check **Stopped (deallocated)**.
- Check for unused disks and public IPs after deleting any VM.
- Check remaining credit regularly.

Use the [monitoring guide](monitoring-setup.md) for service checks. Test private connections before adding alerts. Memory stress on a 1 GiB web VM can kill processes, so test safe limits before the demonstration.
