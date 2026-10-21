## Issue: Proxmox Installer Graphics Failure

### Hardware

- **Server:** HPE ProLiant DL360 G7
- **Hypervisor:** Proxmox VE

### Symptoms

During Proxmox installation, the graphical installer may fail to display correctly or become stuck during boot. This can occur because of graphics driver or framebuffer compatibility issues on older server hardware.

### Workaround

The `nomodeset` kernel parameter can be used to prevent the Linux kernel from enabling kernel mode-setting for graphics during boot.

1. Boot the server from the Proxmox installation USB.
2. At the boot menu, select the installer entry.
3. Press `e` to edit the boot entry.
4. Find the line beginning with `linux`.
5. Add `nomodeset` at the end of that line, separated by a space.
6. Press `Ctrl + X` or `F10` to boot with the modified parameters.

If the graphical installer still fails, try **Install Proxmox VE (Terminal UI)** instead.

### Important Notes

- This is a temporary boot parameter when added through the boot menu; it does not permanently change the installed system's kernel configuration.
- `nomodeset` changes graphics initialization behavior. It is not a general-purpose fix for every display problem.
- Record the exact installer version and observed behavior when reproducing the issue.

### References

- [Proxmox VE Installation Documentation](https://pve.proxmox.com/pve-docs/pve-installation.html)
