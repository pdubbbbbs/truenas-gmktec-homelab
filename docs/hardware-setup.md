# Hardware Setup Guide for TrueNAS on GMKTEC Hardware

This guide covers the hardware setup process for installing TrueNAS on GMKTEC mini PC hardware for a homelab environment.

## GMKTEC Hardware Specifications

### Typical GMKTEC Mini PC Specifications
GMKTEC offers various mini PC models with different configurations. Below are the specifications for a typical model suitable for TrueNAS:

- **Processor**: Intel Celeron/Core i3/i5/i7 (depending on model)
- **Memory**: Support for up to 64GB DDR4 RAM (2 SODIMM slots)
- **Storage**:
  - M.2 NVMe SSD slot for boot drive
  - SATA ports for data drives
  - Options for external storage via USB 3.0/3.1
- **Network**: Dual Gigabit Ethernet ports
- **Expansion**: USB 3.0/3.1 ports, HDMI output
- **Power**: Typically 12-19V DC input, low power consumption

### Recommended Configuration for TrueNAS
For optimal performance with TrueNAS, the following configuration is recommended:

- **Processor**: Intel Core i3 or better
- **Memory**: Minimum 16GB DDR4 RAM (8GB absolute minimum, 32GB+ recommended for ZFS)
- **Boot Drive**: 16GB+ USB drive or small SSD for TrueNAS installation
- **Storage Drives**: 
  - At least 2 identical drives for redundancy
  - Prefer NAS-rated drives (WD Red, Seagate IronWolf, etc.)
- **Network**: Both Ethernet ports connected for redundancy or link aggregation

## Pre-Installation Hardware Checklist

Before proceeding with the installation, ensure you have:

- [ ] GMKTEC mini PC with adequate RAM installed
- [ ] Power adapter for the mini PC
- [ ] USB drive (16GB+) for TrueNAS installation
- [ ] Storage drives for data
- [ ] Ethernet cables
- [ ] USB keyboard and HDMI monitor (for initial setup)
- [ ] Phillips screwdriver for opening the case (if adding internal drives)
- [ ] Anti-static wrist strap (recommended)
- [ ] Network switch with available ports
- [ ] Internet connection (for downloading TrueNAS)

## Physical Setup Instructions

### Preparing the GMKTEC Hardware

1. **Unbox the GMKTEC mini PC**
   - Remove all packaging materials
   - Verify all components are included

2. **Install or Upgrade RAM (if necessary)**
   - Turn off and unplug the mini PC
   - Remove the bottom panel using a Phillips screwdriver
   - Locate the SODIMM slots
   - Install RAM modules following proper alignment
   - Replace the bottom panel

3. **Install Storage Drives**
   - For internal drives:
     - Open the case following the manufacturer's instructions
     - Mount drives securely in available bays or slots
     - Connect SATA and power cables
     - Close the case
   - For external drives:
     - Connect drives to available USB ports
     - Use powered USB hubs if necessary for multiple drives

### Physical Placement

1. **Choose a suitable location**
   - Well-ventilated area with sufficient airflow
   - Away from direct sunlight and sources of heat
   - Accessible for cable management
   - Near your network equipment

2. **Connect peripherals**
   - Connect keyboard and monitor for initial setup
   - Connect Ethernet cable(s) to your network
   - Connect storage drives
   - Connect power adapter

## Network Connectivity Requirements

### Basic Network Setup

1. **Ethernet Connection**
   - Connect primary Ethernet port to your home network
   - Optional: Connect secondary Ethernet port for redundancy or dedicated network

2. **Network Addressing**
   - Prepare to assign a static IP address for your TrueNAS server
   - Note your network's:
     - IP address range
     - Subnet mask
     - Default gateway
     - DNS server addresses

3. **Network Access Requirements**
   - Ensure firewall allows access to the TrueNAS web interface (typically port 80/443)
   - Plan for file sharing protocols:
     - SMB (TCP ports 139, 445)
     - NFS (TCP/UDP ports 111, 2049, 4045-4048)
     - iSCSI (TCP port 3260)

### Advanced Network Considerations

- **Link Aggregation**: Consider configuring both Ethernet ports in LACP mode for increased bandwidth
- **VLAN Support**: Plan for VLAN tagging if used in your network
- **Jumbo Frames**: Consider enabling jumbo frames (MTU 9000) for improved performance if your network supports it

## BIOS/UEFI Configuration Recommendations

Before installing TrueNAS, configure the BIOS/UEFI with these recommended settings:

1. **Accessing BIOS/UEFI**
   - Power on the GMKTEC mini PC
   - Press the appropriate key during boot (usually Delete, F2, or F10)

2. **General Settings**
   - Set date and time correctly
   - Enable virtualization support (VT-x/AMD-V)
   - Enable AES-NI (if available)

3. **Boot Settings**
   - Set boot priority to USB first (for installation)
   - Disable Secure Boot
   - Set to UEFI boot mode (preferred for TrueNAS SCALE) or Legacy mode (for TrueNAS CORE)

4. **Power Management**
   - Disable sleep/hibernation features
   - Set "Power on after power failure" or equivalent option
   - Disable power-saving features that might affect drive availability

5. **Storage Settings**
   - Set SATA controller to AHCI mode
   - Disable onboard RAID if present (TrueNAS will handle software RAID)

6. **Save and Exit**
   - Save changes and exit BIOS/UEFI
   - System should reboot and be ready for TrueNAS installation

## Next Steps

After completing the hardware setup, you're ready to proceed with installing TrueNAS. Please continue to the [Installation Guide](installation-guide.md) for step-by-step instructions.

