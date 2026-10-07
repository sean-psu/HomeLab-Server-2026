# Homelab Network & Proxmox Cluster Configs

This repo has the two switch configs I wrote for a homelab server project we built. The idea was to set up something close to a small corporate network at home while staying separate from the main network in the house.

## The project

The core of the network is a Cisco Enterprise ISR router and a Cisco Catalyst PoE+ managed switch. Residential traffic is isolated from our hypervisor environments with VLANs, and the edge devices run a double NAT so the homelab sits behind its own gateway, separate from the main home internet.

The compute side is a bare-metal Proxmox VE cluster running a mix of VMs and Linux containers (LXCs). Other things running in the project:

- **NGINX reverse proxy** for mapping internal services
- **Inbound and outbound VPN tunnels** to keep sensitive traffic separate and handle external application traffic
- **A custom Discord bot** hosted on the cluster, running across roughly 100 servers, with DDClient handling dynamic DNS
- **A NAS on its own separate hardware** for central storage and redundancy
- **An iperf3 server** for baselining internal throughput and tracking down performance bottlenecks

This repo only covers the switch side. The router, proxy, VPN, bot, and NAS configs aren't included.

## What I did

I wrote the two switch configs in this repo: the VLAN layout, the access ports, and the port-channels that connect the Proxmox nodes and the router.

## VLANs

| VLAN | Name | What it's for |
|------|------|---------------|
| 10 | Virtual | VM and container traffic |
| 20 | Residential / Workstation | Day-to-day workstation traffic, kept away from the hypervisors |
| 50 | General Access | Regular access ports for client devices |
| 100 | ServerHardware | The physical Proxmox hosts |
| 999 | Management | Switch management |

## Server links

Each Proxmox node has two NICs bundled into a port-channel, trunking VLANs 10, 100, and 999:

| Node | Port-channel | Member ports |
|------|--------------|--------------|
| Server0 | Po1 | Gi1, Gi13 |
| Server1 | Po2 | Gi3, Gi15 |
| Server2 | Po3 | Gi4, Gi16 |
| Server3 | Po4 | Gi6, Gi18 |

The router connects over **Port-channel 8** (Gi27 and Gi28), which also trunks VLANs 10, 100, and 999.

## What's in this repo

- `message.txt`: Config for the Catalyst switch. It has the access ports for VLAN 50 (with PortFast) and VLAN 20, PVST spanning tree, the 10G ports, and a dot1q trunk on Gi1/0/45 carrying VLANs 10, 100, and 999.
- `running-config.txt`: Config for the second switch that the servers plug into. It has the VLAN database, the four server port-channels, the router port-channel, and the management VLAN. The hostname, credentials, and management IP are redacted.

## Notes

- PortFast is on for the edge access ports so devices come up quickly without waiting on spanning tree.
- The port-channels give the servers and the router redundancy and extra bandwidth.
- VLAN separation keeps workstation, VM, server, and management traffic apart.
