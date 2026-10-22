# CEH Cybersecurity Lab on Proxmox VE

## 1. Project Overview

This project documents the deployment of a virtualized cybersecurity lab using Proxmox VE on an HPE ProLiant DL360 G7 physical server.

The lab includes Kali Linux, OWASP Broken Web Applications, Metasploitable 2, Windows 10, and Windows Server 2012 R2.

The objective is to build a practical environment for learning virtualization, system administration, network security, vulnerability assessment, and penetration testing without requiring separate physical computers for each operating system.

## 2. Physical Infrastructure

| Component | Configuration |
|---|---|
| Physical server | HPE ProLiant DL360 G7 |
| Hypervisor | Proxmox VE 9.2.0 |
| Hostname | `pve` |
| Management IP | `192.168.10.30` |
| Web interface | `https://192.168.10.30:8006` |
| Virtual network bridge | `vmbr0` |
| ISO storage | `local` |
| VM disk storage | `local-lvm` |

## 3. Virtual Machine Inventory

### VM 100 — Kali Linux

| Setting | Value |
|---|---|
| VM ID | 100 |
| Name | `kali-linux` |
| Virtual disk | 60 GB |
| Memory | 6016 MB |
| Status at last inventory check | Stopped |

**Purpose:**

- Network reconnaissance
- Vulnerability assessment
- Penetration testing
- Security tool experimentation
- Network service enumeration

Kali Linux is used as the primary security testing environment for authorized exercises against the lab's target systems.

### VM 101 — OWASP Broken Web Applications

| Setting | Value |
|---|---|
| VM ID | 101 |
| Name | `OWASP-BWA` |
| Virtual disk | 8 GB |
| Memory | 1504 MB |
| Status at last inventory check | Stopped |

**Purpose:**

- Web application security testing
- Identifying common web vulnerabilities
- Practicing security assessment
- Learning web application attack and defense concepts

This virtual machine provides deliberately vulnerable web applications for controlled security exercises.

### VM 102 — Metasploitable 2

| Setting | Value |
|---|---|
| VM ID | 102 |
| Name | `Metasploitable2` |
| Virtual disk | 8 GB |
| Memory | 1504 MB |
| Status at last inventory check | Stopped |

**Purpose:**

- Service enumeration
- Vulnerability identification
- Network security assessment
- Controlled penetration testing exercises

Metasploitable 2 serves as an intentionally vulnerable target for learning practical security testing techniques.

### VM 103 — Windows 10

| Setting | Value |
|---|---|
| VM ID | 103 |
| Name | `windows-10` |
| Operating system | Windows 10 x64 |
| Virtual CPU | 4 cores |
| Memory | Approximately 5 GB |
| Planned virtual disk | 64 GB |
| Network bridge | `vmbr0` |
| Installation status | Reported installed |

**Purpose:**

- Windows client administration
- Windows networking exercises
- Client-side security concepts
- Interaction with other lab machines
- Practicing Windows-based security scenarios

The Windows 10 installation media was uploaded to Proxmox storage. WinSCP over SFTP was selected as an alternative method for transferring ISO files after a browser-based upload encountered an error.

The final virtual disk capacity and hardware allocation should be verified against the actual VM configuration.

### VM 104 — Windows Server 2012 R2

| Setting | Value |
|---|---|
| VM ID | 104 |
| Name | `windows-server-2012` |
| Operating system | Windows Server 2012 R2 Standard |
| Virtual CPU | 4 cores, 1 socket |
| Memory | Approximately 6.05 GiB allocated |
| Virtual disk | 60 GB |
| Disk format | QCOW2 |
| BIOS | SeaBIOS |
| Machine type | `pc-i440fx-11.0+pve2` |
| SCSI controller | VirtIO SCSI single |
| Network adapter | Intel E1000 |
| Network bridge | `vmbr0` |
| Installation status | Reported installed |

Installation media recorded in Proxmox:

`2012R2STD.ENU.MAY2022.iso`

**Purpose:**

- Windows Server administration
- Server networking
- User and group management
- Learning Windows Server roles
- Active Directory fundamentals after the required roles and domain configuration are completed
- Practicing client-server interactions

The VM hardware specifications were recorded from the Proxmox interface. The operating system installation was confirmed by the project owner.

## 4. Storage Configuration

Proxmox uses separate storage locations for installation media and virtual machine disks.

- `local`: Directory-based storage for ISO images and other supported file content.
- `local-lvm`: Thin-provisioned storage for virtual machine disks.

Storage capacity was inspected using:

```bash
pvesm status
```

Filesystem capacity was checked using:

```bash
df -h
```

The virtual machine inventory was inspected using:

```bash
qm list
```

Individual VM configurations can be inspected using:

```bash
qm config 100
qm config 101
qm config 102
qm config 103
qm config 104
```

### Storage Snapshot — October 22, 2026

| Storage | Total | Used | Available | Usage |
|---|---:|---:|---:|---:|
| `local` | Approximately 93.9 GiB | Approximately 48.1 GiB | Approximately 41.0 GiB | 51.24% |
| `local-lvm` | Approximately 337.5 GiB | Approximately 6.1 GiB | Approximately 331.7 GiB | 1.80% |

These values represent an earlier storage inspection and may change as virtual machines are installed, started, and used.

Thin-provisioned virtual disk capacity is not necessarily equal to actual physical storage consumption.

## 5. ISO Image Management

ISO images are stored in the following directory:

```text
/var/lib/vz/template/iso/
```

When browser-based uploads fail, WinSCP can be used to transfer installation media through SFTP.

Connection settings:

- Protocol: SFTP
- Host: Proxmox management IP
- Port: 22
- Username: `root`
- Destination directory: `/var/lib/vz/template/iso/`

To verify uploaded files:

```bash
ls -lh /var/lib/vz/template/iso/
```

Uploaded ISO images can be selected through:

`local (pve) → ISO Images`

## 6. Virtual Networking and Security

The virtual machines use the Proxmox network bridge `vmbr0`.

Intentionally vulnerable systems, including Metasploitable 2 and OWASP Broken Web Applications, should be isolated from untrusted networks whenever possible.

Recommended precautions:

- Use a dedicated lab network for vulnerable targets.
- Avoid exposing vulnerable virtual machines directly to the public internet.
- Create snapshots before potentially destructive experiments.
- Perform security testing only on authorized systems.
- Keep outdated operating systems isolated from production systems and untrusted networks.

Windows Server 2012 R2 is an obsolete operating system and should be used only in a controlled lab environment.

## 7. Planned Learning Exercises

The environment is intended to support the following learning activities:

1. Linux and Windows system administration.
2. Network discovery and service enumeration.
3. Vulnerability assessment against intentionally vulnerable systems.
4. Web application security testing.
5. Windows client and server networking.
6. Windows Server roles and Active Directory fundamentals.
7. Virtual machine lifecycle management.
8. Storage capacity monitoring and troubleshooting.
9. Documentation of technical configurations and test results.
10. Version control and project documentation using Git and GitHub.

## 8. Maintenance and Verification

The following commands are useful for inspecting the environment:

```bash
pveversion -v
pvesm status
df -h
qm list
qm config 100
qm config 101
qm config 102
qm config 103
qm config 104
```

Configuration changes, networking updates, storage adjustments, and troubleshooting results should be documented as the lab evolves.

## 9. Project Status

The following virtual machines have been created:

- Kali Linux
- OWASP Broken Web Applications
- Metasploitable 2
- Windows 10
- Windows Server 2012 R2

Windows 10 and Windows Server 2012 R2 are reported as installed.

During the last recorded inventory check, Kali Linux, OWASP Broken Web Applications, and Metasploitable 2 were stopped. VM power states may change during subsequent lab exercises.

The environment is being developed as a reusable platform for virtualization practice and cybersecurity training.

## 10. Learning Outcomes

This project demonstrates practical experience with:

- Proxmox VE virtualization
- Virtual machine creation and hardware configuration
- ISO image management and SFTP file transfers
- Virtual disk storage and capacity monitoring
- Linux command-line administration
- Windows client and server virtualization
- Preparation of a cybersecurity lab
- Controlled vulnerability assessment
- Technical documentation using GitHub

The repository will be updated as additional lab exercises, configuration changes, and troubleshooting procedures are completed.
