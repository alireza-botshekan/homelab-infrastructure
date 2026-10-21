# HPE ProLiant DL360 G7 Homelab Infrastructure

A personal homelab project built on an HPE ProLiant DL360 G7 server running Proxmox VE. This repository documents the hardware, virtualization environment, network configuration, storage, and troubleshooting process.

## Hardware

- **Server:** HPE ProLiant DL360 G7
- **CPU:** Intel Xeon X5650 — 6 cores / 12 threads
- **Memory:** 16 GB DDR3 ECC RAM
- **Storage controller:** HP Smart Array P410i / G6 family
- **Storage:** Approximately 465.7 GB logical volume
- **Virtualization:** Proxmox VE
- **Management:** HP iLO 3

*Hardware and storage details are documented in [`docs/hardware-specifications.md`](docs/hardware-specifications.md) and [`docs/storage-and-raid.md`](docs/storage-and-raid.md).*

## Network

The current Proxmox management configuration uses a static IP address and a Linux bridge.

- **Proxmox IP:** `192.168.10.30/24`
- **Gateway:** `192.168.10.1`
- **Bridge:** `vmbr0`
- **Physical interface:** `nic0`

See [`docs/network-configuration.md`](docs/network-configuration.md) for details.

## Documentation

| Document | Description |
|---|---|
| [Hardware Specifications](docs/hardware-specifications.md) | Server hardware and system resources |
| [Proxmox Installation](docs/proxmox-installation.md) | Hypervisor installation and configuration |
| [Network Configuration](docs/network-configuration.md) | Network interfaces, bridge, and routing |
| [Storage and RAID](docs/storage-and-raid.md) | Storage layout and RAID notes |
| [Services and Virtual Machines](docs/services-and-virtual-machines.md) | Virtualization and planned services |
| [Troubleshooting](docs/troubleshooting.md) | Issues, diagnosis, and solutions |
| [iLO Remote Management](docs/ilo-remote-management.md) | Accessing legacy iLO 3 through a compatible browser environment |

## Project Goals

- Build and maintain a self-hosted virtualization environment.
- Learn server administration and Linux system management.
- Document network and storage configuration.
- Practice infrastructure troubleshooting and technical documentation.

## Repository Structure

```text
.
├── README.md
├── docs/
├── screenshots/
└── scripts/
```

## Status

The project is under active development. Documentation is being expanded as configurations are verified and services are deployed.

## Disclaimer

This repository is for educational and homelab purposes. Sensitive information such as credentials, private keys, and confidential network details should never be committed.
