# Hardware Specifications

## Overview

This document describes the hardware configuration of my HPE
ProLiant DL360 G7 home lab server.

The server is used for hands-on learning in virtualization,
Linux administration, networking, storage management, and
infrastructure services.

## Server Hardware

| Component | Specification |
|---|---|
| Server Model | HPE ProLiant DL360 G7 |
| CPU | Intel Xeon X5650 |
| CPU Architecture | x86_64 |
| CPU Cores | 6 |
| CPU Threads | 12 |
| Base Clock | 2.67 GHz |
| Virtualization | Intel VT-x |
| Usable Memory Reported by Linux | 15 GiB |
| Storage Controller | HP Smart Array P410i |
| Logical Disk | 465.7 GB |
| Remote Management | HP iLO 3 |
| iLO Firmware | 1.94 |

## Software Environment

| Component | Version / Details |
|---|---|
| Hypervisor | Proxmox VE |
| Proxmox Version | 9.2.0 |
| Linux Kernel | 7.0.14-25-pve |
| Base Distribution | Debian GNU/Linux 13 (Trixie) |
| Container Management | Docker and Portainer |

## Storage Configuration

The operating system detects a logical disk of approximately
465.7 GB exposed by the storage controller.

The current disk layout includes:

- EFI/boot-related partition
- 1 GB EFI system partition
- LVM physical volume
- 8 GB swap logical volume
- 96 GB root logical volume
- Approximately 338 GB LVM thin-provisioned data storage

The number, model, and capacity of the underlying physical
drives, as well as the RAID level, remain to be verified.

## Network Interfaces

Linux currently reports the following interfaces:

| Interface | Link State |
|---|---|
| nic0 | UP |
| nic1 | DOWN |
| nic2 | DOWN |
| nic3 | DOWN |
| vmbr0 | UP |
| wls1 | DOWN |

`vmbr0` is the active Proxmox Linux bridge. The detailed
network topology and addressing configuration are documented
separately in `network-configuration.md`.

## Intended Use Cases

- Virtual machine deployment and management
- Linux server administration
- Network configuration and troubleshooting
- RAID and storage management
- Containerized services using Docker
- Infrastructure monitoring and automation

## Hardware Details To Verify

- Total installed RAM and DIMM configuration
- Physical drive models and capacities
- RAID level and logical drive configuration
- Network adapter models and link speeds
- Power supply configuration

## Notes

Specifications are based on command output collected from
the running Proxmox host. Memory shown by Linux may differ
slightly from the physically installed capacity.

This document will be updated as the hardware and services
are inspected and configured.

## RAID Configuration

| Property | Value |
|---|---|
| RAID Controller | HP Smart Array P410i / G6 family |
| RAID Level | RAID 0 |
| Logical Volume Size | Approximately 465.7 GB |
| Physical Drive Details | To be documented |

### RAID 0 Considerations

RAID 0 stripes data across multiple drives to improve performance
and combine their capacity. It provides no fault tolerance:
failure of any member drive can make the entire array unavailable.

Important data should be backed up to a separate physical device
or another independent system.
