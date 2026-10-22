# Metasploitable 2 Deployment on Proxmox VE

## Overview

This document describes the process of preparing and attaching a Metasploitable 2 virtual disk to a Proxmox VE virtual machine. It covers disk conversion, storage configuration, troubleshooting, and VM configuration.

The deployment was performed on an HPE ProLiant DL360 G7 server running Proxmox VE 9.2.

## Environment

- **Hypervisor:** Proxmox VE 9.2
- **Server:** HPE ProLiant DL360 G7
- **RAID Controller:** HP Smart Array P410i
- **Storage backend:** LVM-thin (`local-lvm`) and directory storage (`local`)
- **Guest OS:** Metasploitable 2
- **VM ID:** 102
- **Virtual disk size:** 8 GiB
- **RAM:** 1504 MiB
- **CPU:** 1 core
- **Network bridge:** `vmbr0` (initial configuration)

## 1. Initial VMDK Import Attempt

The original VMDK image was stored at:

```text
/var/lib/vz/import/Metasploitable.vmdk
```

The image was inspected using:

```bash
qemu-img info /var/lib/vz/import/Metasploitable.vmdk
```

The image was identified as a VMDK file with an 8 GiB virtual capacity and approximately 1.79 GiB of allocated data.

The initial import was attempted using:

```bash
qm importdisk 102 /var/lib/vz/import/Metasploitable.vmdk local-lvm
```

### Troubleshooting

The import process became stuck for more than 20 minutes. The `qemu-img` process entered the uninterruptible sleep state (`D`), waiting for disk I/O completion.

Useful diagnostic commands included:

```bash
ps -o pid,stat,etime,wchan:30,cmd -p 2573,2602
```

```bash
cat /proc/2602/io
cat /proc/pressure/io
cat /sys/block/sda/inflight
```

```bash
pvesm status
lvs -a -o lv_name,lv_size,data_percent
dmsetup status
```

The investigation showed:

- Both Proxmox storage backends remained active.
- Storage capacity was not exhausted.
- I/O pressure was unusually high.
- The import process stopped making measurable progress.
- No explicit disk I/O error was found in the examined kernel logs.

The exact underlying cause was not conclusively identified.

After a controlled reboot, the stuck `qemu-img` process was no longer present.

## 2. Validate the Source Image

The source image was checked:

```bash
qemu-img check /var/lib/vz/import/Metasploitable.vmdk
```

Output:

```text
No errors were found on the image.
```

This confirmed that the image passed the available image consistency check.

## 3. Convert VMDK to QCOW2

To avoid repeating the same import operation into `local-lvm`, the image was converted to QCOW2 and stored on directory-based storage.

```bash
qemu-img convert -p -f vmdk -O qcow2 \
  /var/lib/vz/import/Metasploitable.vmdk \
  /var/lib/vz/import/Metasploitable2.qcow2
```

The conversion completed successfully at 100%.

The resulting image was verified:

```bash
qemu-img info /var/lib/vz/import/Metasploitable2.qcow2
```

Key results:

```text
file format: qcow2
virtual size: 8 GiB
disk size: 1.79 GiB
corrupt: false
```

## 4. Place the Disk in the VM Image Directory

The standard image directory for VM 102 was created:

```bash
mkdir -p /var/lib/vz/images/102
```

The converted image was moved into that directory:

```bash
mv /var/lib/vz/import/Metasploitable2.qcow2 \
  /var/lib/vz/images/102/vm-102-disk-0.qcow2
```

## 5. Attach the Disk to the VM

The disk was attached to VM 102:

```bash
qm set 102 --scsi0 local:102/vm-102-disk-0.qcow2
```

The boot order was configured:

```bash
qm set 102 --boot order=scsi0
```

The configuration was verified with:

```bash
qm config 102
```

The resulting configuration included:

```text
boot: order=scsi0
cores: 1
memory: 1504
name: Metasploitable2
ostype: l26
scsi0: local:102/vm-102-disk-0.qcow2,size=8G
scsihw: virtio-scsi-single
```

## 6. Storage Configuration Issue

The first attempt to start the VM failed with:

```text
storage 'local' does not support content-type 'images'
```

The storage configuration at `/etc/pve/storage.cfg` showed:

```text
dir: local
        path /var/lib/vz
        content iso,vztmpl,backup,import
```

The `local` storage did not include the `images` content type, which is required for VM disk images.

The proposed correction was:

```bash
pvesm set local --content iso,vztmpl,backup,import,images
```

**Status:** This configuration change and the subsequent VM startup have not yet been verified in the recorded session.

## 7. Next Steps

After enabling the `images` content type:

1. Confirm the storage configuration.
2. Start the VM:

   ```bash
   qm start 102
   ```

3. Check its status:

   ```bash
   qm status 102
   ```

4. Open the VM console in the Proxmox web interface.
5. Confirm that Metasploitable 2 boots successfully.
6. Configure an isolated virtual network before performing security exercises.

## Security Considerations

Metasploitable 2 is intentionally vulnerable and must be treated as an untrusted lab machine.

- Do not expose it directly to the public internet.
- Avoid connecting it to the normal home or production LAN.
- Prefer an isolated Proxmox bridge without a physical network interface.
- Use a separate lab network for Kali Linux and other authorized testing machines.

## Lessons Learned

- Validate virtual disk images before deployment.
- Monitor I/O state and kernel logs when an import stops progressing.
- A lack of storage capacity errors does not rule out a storage I/O problem.
- QCOW2 conversion to directory-based storage provided an alternative to the stalled initial import.
- Proxmox storage content types must be configured appropriately before attaching and using VM disk images.
- Verify VM startup and network isolation before considering the deployment complete.
