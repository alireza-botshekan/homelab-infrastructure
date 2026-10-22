# Metasploitable 2 Deployment on Proxmox VE

## 1. Project Overview

This project documents the deployment of Metasploitable 2, an intentionally vulnerable Linux virtual machine, on a Proxmox VE home lab.

The goal was to deploy the virtual machine, troubleshoot disk import and boot issues, configure its virtual disk, and verify that its web services were accessible from a browser.

**Environment:**
- Hypervisor: Proxmox VE 9.2
- Virtual machine: Metasploitable 2
- VM ID: `102`
- Virtual disk format: QCOW2
- Virtual disk interface: IDE
- Network: Proxmox Linux bridge (`vmbr0`)

## 2. Objectives

- Deploy Metasploitable 2 on Proxmox VE.
- Import and convert a VMDK virtual disk.
- Troubleshoot virtual disk import and boot problems.
- Configure the VM's boot device and storage.
- Verify successful startup and HTTP access.
- Document the deployment for future reference.

## 3. Virtual Disk Preparation

The original Metasploitable 2 virtual disk was provided in VMDK format.

The disk was checked before conversion:

```bash
qemu-img check /var/lib/vz/import/Metasploitable.vmdk
```

The check completed without reporting disk errors.

### Convert VMDK to QCOW2

The disk was converted to QCOW2 format:

```bash
qemu-img convert -p -f vmdk -O qcow2 \
  /var/lib/vz/import/Metasploitable.vmdk \
  /var/lib/vz/import/Metasploitable2.qcow2
```

The resulting image was verified:

```bash
qemu-img info /var/lib/vz/import/Metasploitable2.qcow2
```

The converted image reported a virtual size of 8 GiB and a disk size of approximately 1.79 GiB.

### Place the Disk in the VM Image Directory

The virtual disk was moved to the directory associated with VM 102:

```bash
mkdir -p /var/lib/vz/images/102

mv /var/lib/vz/import/Metasploitable2.qcow2 \
  /var/lib/vz/images/102/vm-102-disk-0.qcow2
```

## 4. Virtual Machine Configuration

The VM was configured with 1 CPU core and approximately 1.5 GiB of RAM.

The disk was attached using the IDE interface:

```bash
qm set 102 --ide0 local:102/vm-102-disk-0.qcow2
qm set 102 --boot order=ide0
```

The IDE interface was selected after the initial SCSI configuration resulted in a boot failure.

## 5. Troubleshooting

### Issue 1: Virtual Disk Import Stalled

The initial disk import to `local-lvm` did not complete promptly and was accompanied by high disk I/O activity.

The system was restarted to recover from the stalled operation. The disk was subsequently converted to QCOW2 and stored on directory-based storage.

### Issue 2: Root Filesystem Not Found

The first boot attempt displayed an error similar to:

```text
ALERT! /dev/mapper/metasploitable-root does not exist.
Dropping to a shell!
```

The disk was detached from the SCSI interface and attached using IDE instead. The VM was then configured to boot from the IDE disk.

### Issue 3: Local Storage Did Not Support VM Images

Proxmox reported:

```text
storage 'local' does not support content-type 'images'
```

The directory-based `local` storage configuration did not include `images` in its supported content types.

The configuration was updated:

```bash
pvesm set local --content iso,vztmpl,backup,import,images
```

After this change, Proxmox could use the `local` storage for the VM disk.

## 6. Verification

The VM started successfully after the disk and storage configuration changes.

The following checks were performed:

- The Metasploitable 2 login console appeared.
- The system startup sequence showed web services starting.
- The VM's network configuration was checked using `ifconfig`.
- The VM's IP address was entered into a web browser on the host laptop.
- The web interface loaded successfully.

### Web Interface Screenshot

The following screenshot documents successful browser access to the Metasploitable 2 web interface.

![Metasploitable 2 Web Interface](../screenshots/metasploitable2/Screenshot%202026-10-22%20055459.png)

[Open the original screenshot](../screenshots/metasploitable2/Screenshot%202026-10-22%20055459.png)

## 7. Security Considerations

Metasploitable 2 is intentionally vulnerable and is designed for security training and penetration-testing practice.

**Network isolation is strongly recommended.**

- Do not expose the VM directly to the public internet.
- Prefer an isolated Proxmox bridge or a dedicated lab network.
- Avoid placing the VM on the normal home LAN while performing security tests.
- Do not use the VM to test systems without explicit authorization.
- Verify the network configuration before beginning penetration-testing exercises.

The VM was initially connected through `vmbr0`. Its isolation should be verified before further security testing.

## 8. Skills Demonstrated

This project demonstrates practical experience with:

- Proxmox VE virtual machine management
- Linux command-line administration
- QEMU disk image inspection and conversion
- VMDK and QCOW2 virtual disk formats
- Virtual disk attachment and boot configuration
- Proxmox storage configuration
- Troubleshooting virtual machine boot failures
- Basic network and web-service verification
- Technical documentation using Markdown and GitHub

## 9. Project Outcome

Metasploitable 2 was successfully deployed on Proxmox VE, and its web interface was verified from a browser.

The deployment required troubleshooting a stalled disk import, a root filesystem boot error, and a Proxmox storage content-type configuration issue.

This project is part of my home lab infrastructure and provides a foundation for isolated penetration-testing and cybersecurity practice.

---

**Repository:** [homelab-infrastructure](https://github.com/alireza-botshekan/homelab-infrastructure)
