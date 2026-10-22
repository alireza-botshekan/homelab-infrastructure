# Metasploitable 2 Deployment on Proxmox VE

## 1. Project Overview

This project documents the deployment of Metasploitable 2 on a home Proxmox VE server to create a controlled environment for cybersecurity learning, CEH practice, and web service testing.

During deployment, the original VMDK image was validated, an initial disk import encountered an I/O stall, and the disk was converted to QCOW2. The virtual disk was then attached to the VM, boot configuration was adjusted, and a storage configuration issue was resolved.

**Final result:** Metasploitable 2 booted successfully, and its web page was accessible from a browser.

## 2. Infrastructure Specifications

| Component | Specification |
|---|---|
| Server | HPE ProLiant DL360 G7 |
| Hypervisor | Proxmox VE 9.2 |
| RAID controller | HP Smart Array P410i |
| VM ID | 102 |
| VM name | Metasploitable2 |
| Guest OS | Metasploitable 2 |
| Memory | 1504 MB |
| CPU | 1 core |
| Virtual disk capacity | 8 GiB |
| Original disk format | VMDK |
| Final disk format | QCOW2 |
| Final storage | `local` |
| Final disk controller | IDE |
| Initial network bridge | `vmbr0` |

## 3. Inspecting the Original Disk

The original virtual disk was located at:

```text
/var/lib/vz/import/Metasploitable.vmdk
```

Its properties were inspected with:

```bash
qemu-img info /var/lib/vz/import/Metasploitable.vmdk
```

The image was identified as a VMDK file with an 8 GiB virtual capacity and approximately 1.79 GiB of allocated data.

An initial image check was performed:

```bash
qemu-img check /var/lib/vz/import/Metasploitable.vmdk
```

Output:

```text
No errors were found on the image.
```

This check reported no errors in the image structures it examined.

## 4. Troubleshooting the Initial Disk Import

The initial attempt used Proxmox's disk import command:

```bash
qm importdisk 102 /var/lib/vz/import/Metasploitable.vmdk local-lvm
```

The operation continued for more than 20 minutes without completing.

The process state and I/O activity were investigated using:

```bash
ps -o pid,stat,etime,wchan:30,cmd -p 2573,2602
```

```bash
cat /proc/2602/io
cat /proc/pressure/io
cat /sys/block/sda/inflight
```

Storage and device-mapper status were also checked:

```bash
pvesm status
lvs -a -o lv_name,lv_size,data_percent
dmsetup status
```

### Findings

- The `qemu-img` process entered the uninterruptible sleep state (`D`), waiting for I/O completion.
- System I/O pressure became unusually high.
- Storage capacity was not exhausted.
- Both `local` and `local-lvm` remained active.
- No explicit `I/O error` was found in the kernel logs examined during troubleshooting.

The precise underlying cause of the stalled import was not conclusively identified.

After a server reboot, the stalled `qemu-img` process was no longer present.

## 5. Converting VMDK to QCOW2

Instead of repeating the original import into `local-lvm`, the disk was converted to QCOW2 and stored on directory-based `local` storage.

```bash
qemu-img convert -p -f vmdk -O qcow2 \
  /var/lib/vz/import/Metasploitable.vmdk \
  /var/lib/vz/import/Metasploitable2.qcow2
```

The conversion completed successfully at 100%.

The resulting image was inspected:

```bash
qemu-img info /var/lib/vz/import/Metasploitable2.qcow2
```

Results:

- **Format:** QCOW2
- **Virtual capacity:** 8 GiB
- **Actual file size:** approximately 1.79 GiB
- **Reported corruption status:** `corrupt: false`

## 6. Moving the Disk to the VM Image Directory

The directory for VM 102 was created:

```bash
mkdir -p /var/lib/vz/images/102
```

The converted image was moved to the standard VM image path:

```bash
mv /var/lib/vz/import/Metasploitable2.qcow2 \
  /var/lib/vz/images/102/vm-102-disk-0.qcow2
```

## 7. Attaching the Disk and Configuring Boot

The disk was initially attached as a SCSI device:

```bash
qm set 102 --scsi0 local:102/vm-102-disk-0.qcow2
```

The boot order was set to use the SCSI disk:

```bash
qm set 102 --boot order=scsi0
```

The VM configuration was checked with:

```bash
qm config 102
```

However, the guest operating system failed to find its root filesystem and dropped into the `initramfs` shell.

The error displayed was:

```text
ALERT! /dev/mapper/metasploitable-root does not exist.
Dropping to a shell!
```

This indicated that the guest could not locate the expected root device during boot.

The VM was shut down, and the disk controller was changed from SCSI to IDE.

First, the SCSI attachment was removed:

```bash
qm set 102 --delete scsi0
```

The disk was then attached as an IDE device:

```bash
qm set 102 --ide0 local:102/vm-102-disk-0.qcow2
```

The boot order was updated:

```bash
qm set 102 --boot order=ide0
```

The final configuration was verified:

```bash
qm config 102
```

Relevant configuration:

```text
boot: order=ide0
ide0: local:102/vm-102-disk-0.qcow2,size=8G
```

## 8. Resolving the Proxmox Storage Configuration Issue

The first attempt to start the VM failed with:

```text
storage 'local' does not support content-type 'images'
```

Inspection of `/etc/pve/storage.cfg` showed that the `local` storage configuration did not include the `images` content type:

```text
dir: local
        path /var/lib/vz
        content iso,vztmpl,backup,import
```

The following command was used to add support for VM images:

```bash
pvesm set local --content iso,vztmpl,backup,import,images
```

After this change, the disk could be used as a VM image on `local` storage.

## 9. Successful Boot Verification

The VM was started using:

```bash
qm start 102
```

Metasploitable 2 booted successfully and displayed its login prompt.

The default login credentials were:

```text
Username: msfadmin
Password: msfadmin
```

The console also displayed successful startup messages for web services, including Apache and Tomcat.

## 10. Verifying Web Access

After logging into the guest operating system, its IP address was checked using:

```bash
ifconfig
```

The VM's IP address was then entered into the laptop's browser:

```text
http://<METASPLOITABLE_IP>
```

The web page opened successfully.

This confirmed that the guest operating system had booted and that HTTP access from the laptop to the VM was working.

**Note:** The exact IP address and a screenshot of the final web page have not been recorded in this document. They can be added as project evidence later.

### Web Interface Screenshot

The following screenshot demonstrates successful HTTP access to the Metasploitable 2 web interface.

![Metasploitable 2 Web Interface](../screenshots/metasploitable2/web-page.png)

## 11. Network Security Considerations

Metasploitable 2 is intentionally vulnerable and should only run in a controlled lab environment.

The VM's initial network interface was connected to `vmbr0`, which may provide access to the physical network.

Before performing security exercises:

- Do not expose the VM directly to the public internet.
- Avoid connecting it to the normal home or production LAN.
- Create an isolated virtual bridge without a physical network interface.
- Use a separate private network if Kali Linux needs to communicate with Metasploitable 2.
- Restrict network access to authorized lab systems.

**Network isolation has not yet been verified as completed.**

## 12. Project Outcome

The following tasks were completed:

- Inspected and validated the original VMDK image.
- Investigated a stalled disk import.
- Converted the disk from VMDK to QCOW2.
- Attached the converted disk to VM 102.
- Changed the virtual disk controller from SCSI to IDE.
- Corrected the Proxmox storage content configuration.
- Booted Metasploitable 2 successfully.
- Accessed its web page from a browser.

### Skills Demonstrated

- Proxmox VE virtual machine management
- Directory and LVM-thin storage management
- Virtual disk conversion with `qemu-img`
- Virtual disk attachment and boot configuration
- Linux boot troubleshooting and `initramfs` diagnosis
- Linux process and I/O diagnostics
- Proxmox storage configuration troubleshooting
- Building a virtual lab for cybersecurity practice

This project provided practical experience with VM deployment, disk-format conversion, storage troubleshooting, and guest OS boot configuration.

The next step is to isolate the lab network and record the VM's IP address and a screenshot of the working web page.
