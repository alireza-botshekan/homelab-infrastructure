# Network Configuration

## Overview

This document describes the network configuration of an HPE ProLiant DL360 G7 server running Proxmox VE.

## Current Network Configuration

| Property | Value |
|---|---|
| Proxmox management IP | `192.168.10.30/24` |
| Default gateway | `192.168.10.1` |
| Network subnet | `192.168.10.0/24` |
| Active bridge | `vmbr0` |
| Bridge physical port | `nic0` |
| IPv4 configuration | Static |
| Bridge STP | Disabled |
| Bridge forwarding delay | 0 |

## Network Interfaces

| Interface | State | Configuration |
|---|---|---|
| `nic0` | UP | Physical interface attached to `vmbr0` |
| `nic1` | DOWN | Manual |
| `nic2` | DOWN | Manual |
| `nic3` | DOWN | Manual |
| `vmbr0` | UP | Static IP address configured |
| `wls1` | DOWN | Wireless interface reported by the host |

## Linux Bridge

The `vmbr0` bridge is configured with `nic0` as its physical bridge port.

The Proxmox management IP address is assigned to `vmbr0`, rather than directly to `nic0`.

## Routing

The default route is configured through `192.168.10.1` using `vmbr0`.

The connected network is `192.168.10.0/24`.

## Network Configuration File

The configuration is stored in `/etc/network/interfaces`.

Relevant configuration:

```text
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.10.30/24
        gateway 192.168.10.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0

iface nic1 inet manual
iface nic2 inet manual
iface nic3 inet manual
```

## Security Notes

- Do not publish passwords, authentication tokens, or private keys.
- Review screenshots for sensitive information before committing them.
- The iLO management IP address has not yet been verified.

## Future Improvements

- Verify the current iLO IP address.
- Document the router and physical network topology.
- Add a network topology diagram.
- Document additional VLANs or services if configured.