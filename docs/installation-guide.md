# TrueNAS Installation Guide for GMKTEC Hardware

This guide provides step-by-step instructions for installing TrueNAS on GMKTEC mini PC hardware for your homelab environment.

## Prerequisites and Required Materials

Before starting the installation process, ensure you have completed the [hardware setup](hardware-setup.md) and have the following:

### Required Materials
- GMKTEC mini PC with hardware properly configured
- USB flash drive (16GB minimum, preferably 32GB)
- Keyboard and monitor connected to the GMKTEC system
- Internet connection for downloading TrueNAS
- Another computer with internet access and USB ports for creating the boot media
- Network cable connected to your local network
- All storage drives connected and ready

### Required Knowledge
- Basic understanding of computer hardware
- Familiarity with BIOS/UEFI settings
- Basic networking concepts
- Understanding of disk partitioning and storage concepts

## TrueNAS Version Selection: CORE vs SCALE

TrueNAS offers two main versions, and you'll need to decide which one best fits your needs:

### TrueNAS CORE
- Based on FreeBSD
- Mature and stable platform
- Excellent ZFS implementation
- Traditional NAS functionality
- Supports plugins via jails
- **Recommended for**: Users who primarily need file sharing and storage features

### TrueNAS SCALE
- Based on Linux (Debian)
- Newer with additional features
- Native Docker container support
- Kubernetes integration
- App catalog with easier application deployment
- **Recommended for**: Users who want container support and advanced applications

### Recommendation for GMKTEC Hardware
For GMKTEC mini PC hardware:
- **TrueNAS SCALE** is generally recommended for better hardware compatibility with modern Intel CPUs and potentially better driver support
- If you plan to run Docker containers or Kubernetes workloads, SCALE is the clear choice
- If you only need file sharing and traditional NAS features, either version will work well

## USB Boot Media Creation Process

### 1. Download TrueNAS Image
1. Visit the [TrueNAS download page](https://www.truenas.com/download-truenas-core/)
2. Download either:
   - TrueNAS CORE (if you chose the FreeBSD version)
   - TrueNAS SCALE (if you chose the Linux version)
3. Verify the checksum of the downloaded ISO file (optional but recommended)

### 2. Create Bootable USB Drive

#### On Windows:
1. Download and install [Rufus](https://rufus.ie/) or [balenaEtcher](https://www.balena.io/etcher/)
2. Insert your USB drive (note that it will be completely erased)
3. Open Rufus or balenaEtcher
4. Select the TrueNAS ISO file you downloaded
5. Select your USB drive as the target
6. Click "Start" or "Flash" and wait for the process to complete

#### On macOS:
1. Download and install [balenaEtcher](https://www.balena.io/etcher/)
2. Insert your USB drive (note that it will be completely erased)
3. Open balenaEtcher
4. Select the TrueNAS ISO file you downloaded
5. Select your USB drive as the target
6. Click "Flash" and wait for the process to complete

#### On Linux:
1. Download and install [balenaEtcher](https://www.balena.io/etcher/) or use the `dd` command
2. Insert your USB drive (note that it will be completely erased)
3. For balenaEtcher, follow the same steps as macOS
4. For `dd`, use:
   ```bash
   # Replace X with your USB drive letter
   sudo dd if=/path/to/truenas.iso of=/dev/sdX bs=4M status=progress
   ```

### 3. Prepare for Installation
1. Safely eject the USB drive from your computer
2. Insert the USB drive into one of the USB ports on your GMKTEC mini PC
3. Ensure monitor and keyboard are connected
4. Power on the GMKTEC system

## Step-by-Step Installation Instructions

### 1. Boot from USB Drive
1. Power on your GMKTEC mini PC
2. Press the appropriate key to enter the boot menu (typically F12, F10, or ESC)
3. Select the USB drive from the boot menu
4. The TrueNAS installer should load (this may take a few minutes)

### 2. TrueNAS Installation Wizard

#### For TrueNAS CORE:
1. At the main menu, select "Install/Upgrade"
2. Select the destination media where TrueNAS will be installed:
   - For GMKTEC systems, this will typically be the internal SSD or a separate USB drive
   - **IMPORTANT**: Do not select any drives you plan to use for data storage
   - A minimum 16GB drive is recommended for the OS
3. Select "Yes" when asked if you're sure
4. Set a root password when prompted
5. Select the boot method that matches your system (UEFI or BIOS)
6. Wait for the installation to complete
7. When prompted, select "Reboot System" and remove the USB installer

#### For TrueNAS SCALE:
1. Select your language and keyboard layout
2. Select "Install TrueNAS SCALE"
3. Choose the destination drive (as above, select the internal SSD or a separate boot device)
4. Create a root password
5. Select the boot method (UEFI recommended for modern GMKTEC hardware)
6. Wait for the installation to complete
7. When prompted, reboot the system and remove the USB installer

### 3. Initial Boot
1. The system will reboot and start TrueNAS
2. You may see some boot messages as the system initializes
3. Eventually, you'll see the TrueNAS console screen with the IP address of the web interface

## Initial System Configuration

### 1. Accessing the Web Interface
1. From another computer on the same network, open a web browser
2. Enter the IP address shown on the TrueNAS console screen
3. Accept any security certificates (you can properly configure SSL later)
4. Log in with:
   - Username: root
   - Password: (the password you set during installation)

### 2. Initial Setup Wizard
1. Follow the on-screen wizard to configure basic settings:
   - Set your region and timezone
   - Configure network settings (recommended to set a static IP)
   - Set up storage (covered in more detail in the [Configuration Guide](configuration.md))

### 3. Network Configuration
1. Navigate to Network > Global Configuration
2. Set the hostname for your TrueNAS system (e.g., `truenas-gmktec`)
3. Configure DNS servers
4. Navigate to Network > Interfaces
5. Select the active network interface and set:
   - DHCP or Static IP (static recommended)
   - If using static, enter IP address, subnet mask, and gateway

### 4. Storage Configuration
1. Navigate to Storage > Pools
2. Click "Add" to create a new storage pool
3. Follow the wizard to create your storage pool:
   - Select the disks you want to include
   - Choose the appropriate RAID level (mirrors are recommended for 2 drives)
   - Name your pool (e.g., "tank", "data", etc.)
4. Create datasets for your data organization

## Post-Installation Verification Steps

### 1. System Health Check
1. Navigate to Dashboard and verify:
   - All hardware is detected correctly
   - CPU and memory usage appear normal
   - Storage pools are online
   - Network interfaces show correct connection status

### 2. Storage Verification
1. Navigate to Storage > Pools
2. Verify all disks are healthy (should show "ONLINE" status)
3. Run a quick SMART test on each disk:
   - Navigate to Storage > Disks
   - Select each disk and click "Run SMART Test"
   - Choose "Short Test"
4. Create a test file to verify write capabilities:
   - Navigate to Storage > Pools
   - Select your pool and click "Add Dataset"
   - Create a test dataset
   - Navigate to the dataset and upload a test file

### 3. Network Verification
1. Verify connectivity:
   - Ping the TrueNAS system from another computer
   - Ping another device or the internet from TrueNAS (via Shell)
2. Test network speed (optional):
   - Use iperf or a file transfer to test network performance

### 4. Service Configuration
1. Navigate to Services
2. Enable services you plan to use:
   - SMB (for Windows file sharing)
   - NFS (for Linux/Unix file sharing)
   - SSH (for remote administration)
3. Configure sharing permissions (detailed in the [Configuration Guide](configuration.md))

### 5. Update TrueNAS
1. Navigate to System > Update
2. Check for available updates
3. Apply any system updates
4. Reboot if prompted after updates

## Troubleshooting Common Installation Issues

### Boot Issues
- **Problem**: System doesn't boot from USB
  - **Solution**: Check BIOS settings, try a different USB port, recreate the installation media

- **Problem**: Error "No bootable device"
  - **Solution**: Verify boot order in BIOS, ensure installation media is properly created

### Hardware Detection Issues
- **Problem**: Network interfaces not detected
  - **Solution**: Check cable connections, try different network ports, verify drivers

- **Problem**: Drives not detected
  - **Solution**: Check SATA/power connections, verify AHCI mode in BIOS

### Performance Issues
- **Problem**: System feels sluggish
  - **Solution**: Verify adequate RAM (16GB+ recommended), check for disk errors, ensure cooling is adequate

## Next Steps

Now that you've successfully installed TrueNAS on your GMKTEC hardware, proceed to the [Configuration Guide](configuration.md) to set up shares, user accounts, and other important features for your homelab NAS.

