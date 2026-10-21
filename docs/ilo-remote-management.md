\# HP iLO 3 Remote Management on Legacy Browsers



\## Overview



This document describes the process of accessing the HP Integrated Lights-Out 3 (iLO 3) management interface on an HPE ProLiant DL360 G7 server.



The iLO 3 web interface is legacy technology and may not work correctly in modern browsers because of compatibility limitations involving older web technologies, Java components, and remote console requirements.



\## Environment



| Component | Details |

|---|---|

| Server | HPE ProLiant DL360 G7 |

| Remote management | HP iLO 3 |

| iLO firmware | Version 1.94 (previously reported) |

| Legacy browser environment | Windows 7 virtual machine |

| Virtualization software | Oracle VirtualBox |

| Browser used | Internet Explorer |



\## Network Connectivity



Before opening the iLO web interface, the laptop and the iLO management port must have network connectivity.



Two connection methods are possible:



\### Method A: Using a Router



1\. Connect the dedicated iLO Ethernet port to a LAN port on the router.

2\. Connect the laptop to the same local network.

3\. Check whether DHCP is enabled and whether iLO receives an IP address.

4\. Find the iLO address in the router's DHCP client list, if available.

5\. Verify connectivity to the iLO address before opening the web interface.



\### Method B: Direct Laptop Connection



1\. Connect the laptop's Ethernet port directly to the dedicated iLO port.

2\. Identify the current iLO IP address using the server's iLO configuration utility or other available management method.

3\. Configure a compatible static IP address on the laptop.

4\. Ensure that the laptop and iLO addresses belong to the same subnet and do not conflict.



If the current iLO address is unknown, verify it before changing network settings. Do not assume that historical IP addresses are still valid.



\## Accessing iLO from Windows 7



A Windows 7 virtual machine running in Oracle VirtualBox was used as a legacy browser environment.



The intended workflow is:



1\. Configure the virtual machine's network so that it can reach the iLO management interface.

2\. Open Internet Explorer inside Windows 7.

3\. Navigate to the verified iLO IP address.

4\. Sign in using the configured iLO credentials.

5\. Install or enable only the legacy components required for the remote console, using trusted sources and versions compatible with the installed iLO firmware.

6\. Test the remote console and record the result.



The browser interface and the remote console may have different compatibility requirements. Successfully opening the web page does not necessarily mean that the remote console will work.



\## Virtual Machine Networking



The Windows 7 virtual machine must be able to reach the iLO management IP.



Depending on the host network, Oracle VirtualBox networking may require Bridged Adapter or another correctly configured network mode.



Verify the virtual machine's connectivity before troubleshooting the browser itself.



\## Security Considerations



\- Do not expose iLO directly to the public internet.

\- Do not publish iLO credentials, passwords, session cookies, or private keys.

\- Prefer an isolated local management network.

\- Windows 7, Internet Explorer, and legacy browser components are unsupported or insecure for general internet use. Keep the virtual machine isolated from untrusted websites.

\- Download legacy components only from trustworthy sources.



\## Verification Status



The Windows 7 and Internet Explorer approach was used as a compatibility workaround. The exact network setup, required components, and remote console behavior should be documented after each step is verified.



\## Related Documentation



\- \[Network Configuration](network-configuration.md)

\- \[Troubleshooting](troubleshooting.md)

\- \[Hardware Specifications](hardware-specifications.md)



