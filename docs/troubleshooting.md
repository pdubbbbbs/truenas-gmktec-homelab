# TrueNAS Troubleshooting Guide for GMKTEC Homelab

This troubleshooting guide addresses common issues you may encounter while running TrueNAS on GMKTEC hardware in a homelab environment. Each section includes symptoms, diagnostic steps, and solutions.

## Common Hardware Issues Specific to GMKTEC Systems

### Boot Problems

#### System Won't Boot

**Symptoms:**
- System doesn't power on
- System powers on but doesn't display anything
- "No bootable device" error

**Diagnostic Steps:**
1. Check power connections
2. Verify the power LED is illuminated
3. Connect a monitor to check for BIOS messages
4. Check if USB boot devices are recognized

**Solutions:**
- Re-seat power connections
- Try a different power outlet or power adapter
- Reset CMOS (consult your GMKTEC model's manual)
- Re-create bootable USB if booting from USB
- Check boot order in BIOS

#### TrueNAS Boot Errors

**Symptoms:**
- System reaches GRUB menu but fails to boot TrueNAS
- Boot process stops with kernel messages

**Diagnostic Steps:**
1. Note any error messages displayed
2. Enter GRUB menu and attempt to boot with verbose mode
3. Check if hardware has changed since installation

**Solutions:**
- For UEFI boot issues: Disable Secure Boot in BIOS
- For corrupted boot: Boot from TrueNAS installation media and select "Shell" to access recovery tools
- Reinstall TrueNAS to boot device if necessary

### Thermal Issues

**Symptoms:**
- System shuts down unexpectedly
- Fans running at high speed constantly
- Performance throttling due to heat

**Diagnostic Steps:**
1. Check system temperatures in TrueNAS dashboard
2. Verify ambient room temperature
3. Check for dust buildup in vents

**Solutions:**
- Ensure GMKTEC mini PC has adequate airflow around it
- Clean dust from air vents (carefully)
- Place GMKTEC system in a cooler location
- Consider adding external cooling for high-load environments
- Limit CPU-intensive tasks in TrueNAS (adjust service priorities)

### USB Device Issues

**Symptoms:**
- External USB drives disconnect randomly
- USB devices not recognized

**Diagnostic Steps:**
1. Check system logs for USB-related errors
   ```
   dmesg | grep -i usb
   ```
2. Try different USB ports
3. Test with different USB cables

**Solutions:**
- Use powered USB hub for external drives
- Update TrueNAS to latest version for improved USB support
- Avoid USB 3.0 ports if experiencing compatibility issues with certain devices
- Check for USB power saving settings in BIOS and disable them

### Limited SATA Ports

**Symptoms:**
- Cannot connect all desired storage drives

**Solutions:**
- Use USB to SATA adapters for additional drives (with caution)
- Implement network storage expansion via iSCSI or NFS
- Consider a storage controller PCIe card if your GMKTEC model supports expansion

## Network Connectivity and Performance Problems

### Connection Issues

#### Cannot Access Web Interface

**Symptoms:**
- Unable to reach TrueNAS web interface from browser
- "This site can't be reached" or timeout error

**Diagnostic Steps:**
1. Verify physical network connections
2. Check IP address on TrueNAS console
3. Ping TrueNAS IP from another computer
4. Check for IP address conflicts

**Solutions:**
- Re-seat network cables
- Restart networking on TrueNAS:
  ```
  service networking restart
  ```
- Reset network settings via console
- Check router/switch for issues

#### Intermittent Network Dropouts

**Symptoms:**
- Periodic loss of connection to TrueNAS
- File transfers stop unexpectedly

**Diagnostic Steps:**
1. Check network interface statistics in TrueNAS
2. Run continuous ping to test stability
3. Examine network cables for damage
4. Check system logs for network errors

**Solutions:**
- Replace network cables
- Try different network ports on router/switch
- Disable green ethernet/power saving features on network interfaces
- Update network driver through TrueNAS update

### Performance Issues

#### Slow Network Transfer Speeds

**Symptoms:**
- File transfers much slower than expected
- Inconsistent network performance

**Diagnostic Steps:**
1. Test network speed using iperf3
2. Check interface settings (speed/duplex)
3. Monitor network utilization during transfers
4. Verify switch capabilities

**Solutions:**
- Ensure network is set to correct speed (1Gbps)
- Configure jumbo frames (MTU 9000) if supported
- Disable unnecessary network services
- Use CAT6 or better ethernet cables
- Update NIC firmware/drivers

#### WiFi Instead of Ethernet Issues

**Symptoms:**
- Poor performance when clients use WiFi

**Solutions:**
- Ensure optimal WiFi router placement
- Use 5GHz band for better performance
- Consider MoCA or powerline adapters for areas with poor WiFi
- Set up QoS on your router to prioritize NAS traffic

## Storage and ZFS-related Issues

### Pool Status Problems

#### DEGRADED Pool Status

**Symptoms:**
- Storage pool shows DEGRADED status
- Alert notifications about drive errors

**Diagnostic Steps:**
1. Check pool status:
   ```
   zpool status
   ```
2. Identify which drive is having issues
3. Run a SMART test on the problematic drive:
   ```
   smartctl -t long /dev/sdX
   ```

**Solutions:**
- Replace failing drive
- Resilver the pool after drive replacement
- If using external USB drives, check connections and power

#### UNAVAIL Device Errors

**Symptoms:**
- Drive shows as UNAVAIL in pool status
- Data access issues for affected pool

**Diagnostic Steps:**
1. Check if drive is physically connected
2. Examine drive identifiers in `zpool status`
3. Check system logs for drive errors

**Solutions:**
- Reconnect drive if disconnected
- Export pool and import again to detect devices
- If drive failed, replace and let ZFS resilver

### Data Corruption Issues

**Symptoms:**
- Checksum errors in `zpool status`
- Files appear corrupted or unreadable

**Diagnostic Steps:**
1. Run scrub to identify extent of damage:
   ```
   zpool scrub poolname
   ```
2. Check for SMART errors on drives

**Solutions:**
- If redundancy exists, clear errors:
   ```
   zpool clear poolname
   ```
- Restore affected files from backup
- For persistent issues, replace affected drive

### Performance Issues

#### Slow Disk Performance

**Symptoms:**
- Poor read/write speeds
- System feels sluggish
- High disk utilization

**Diagnostic Steps:**
1. Check disk activity and performance:
   ```
   iostat -xm 2
   ```
2. Verify drive health with SMART tests
3. Check pool fragmentation:
   ```
   zpool list -v
   ```

**Solutions:**
- Adjust dataset record size for workload
- Add SSD cache devices (L2ARC) for read-heavy workloads
- Add log devices (SLOG) for sync write workloads
- Check for drive vibration issues in mini PC enclosure
- Consider separating busy datasets to different pools

#### Out of Space Issues

**Symptoms:**
- "No space left on device" errors
- Pool shows high usage

**Diagnostic Steps:**
1. Check actual pool usage:
   ```
   zpool list
   ```
2. Check dataset usage:
   ```
   zfs list -o name,used,avail,refer
   ```
3. Check for snapshots consuming space:
   ```
   zfs list -t snapshot
   ```

**Solutions:**
- Delete unnecessary snapshots
- Increase pool capacity by adding drives
- Implement quotas on datasets
- Move data to external storage

## Service-Specific Troubleshooting (SMB, NFS)

### SMB Issues

#### Cannot Connect to SMB Shares

**Symptoms:**
- "Network path not found" when accessing shares
- Authentication errors when connecting

**Diagnostic Steps:**
1. Verify SMB service is running in TrueNAS
2. Check SMB share configuration
3. Test connectivity from client:
   ```
   smbclient -L //truenas-ip -U username
   ```

**Solutions:**
- Restart SMB service
- Check user permissions and passwords
- Ensure client and server authentication settings match
- Verify network connectivity between client and server

#### Poor SMB Performance

**Symptoms:**
- Slow file transfers via SMB
- High latency when opening files

**Diagnostic Steps:**
1. Check SMB log for errors
2. Monitor network traffic during transfers
3. Test with different SMB protocol versions

**Solutions:**
- Configure SMB to use SMB3 protocol
- Adjust SMB socket options for better performance
- Enable large MTU if network supports it
- Optimize dataset record size for workload

### NFS Issues

#### NFS Mount Failures

**Symptoms:**
- "mount.nfs: access denied" errors
- Timeouts when mounting

**Diagnostic Steps:**
1. Verify NFS service is running
2. Check NFS share permissions and allowed networks
3. Test connectivity with `showmount -e truenas-ip`

**Solutions:**
- Check client IP is in allowed ranges
- Verify network connectivity
- Restart NFS service
- Ensure NFS versions match between client and server

#### NFS Performance Issues

**Symptoms:**
- Slow read/write performance
- High CPU usage during transfers

**Diagnostic Steps:**
1. Check NFS version being used
2. Monitor system resources during transfers
3. Test with different mount options

**Solutions:**
- Use NFSv4 instead of NFSv3 when possible
- Configure optimal read/write sizes
- Adjust number of NFS server processes
- Use async mounts for non-critical data

## System Performance Issues

### High CPU Usage

**Symptoms:**
- System performance degradation
- Web interface becomes sluggish
- Top processes consuming CPU according to dashboard

**Diagnostic Steps:**
1. Identify top processes using CPU:
   ```
   top -o cpu
   ```
2. Check for scheduled tasks running
3. Monitor CPU temperature

**Solutions:**
- Limit concurrent operations
- Adjust schedules for CPU-intensive tasks
- Check for runaway processes and restart them
- Reduce the number of enabled plugins/jails/VMs
- Consider CPU upgrade if GMKTEC model allows

### Memory Issues

**Symptoms:**
- System shows high memory usage
- Swapping activity detected
- Services restart unexpectedly

**Diagnostic Steps:**
1. Check memory usage:
   ```
   top -o res
   ```
2. Monitor ZFS ARC size:
   ```
   arc_summary
   ```
3. Check for memory leaks in services

**Solutions:**
- Limit ZFS ARC size in System > Advanced
- Reduce number of running services
- Upgrade RAM if GMKTEC model allows
- Adjust VM/jail memory limits
- Restart services showing memory growth

### Disk I/O Bottlenecks

**Symptoms:**
- High disk utilization
- Operations timing out
- Poor overall system responsiveness

**Diagnostic Steps:**
1. Identify high I/O processes:
   ```
   iostat -xm 2
   ```
2. Check disk queue lengths
3. Monitor for specific read vs write issues

**Solutions:**
- Schedule intensive operations during off-peak times
- Add SSD cache/log devices
- Separate busy datasets to different physical disks
- Consider faster disk upgrade
- Use external USB 3.1/3.2 instead of USB 3.0 for better performance

## Update and Upgrade Problems

### Failed Updates

**Symptoms:**
- Update process fails with errors
- System becomes unbootable after update

**Diagnostic Steps:**
1. Check update logs
2. Verify system requirements for update
3. Check available space on boot device

**Solutions:**
- Boot into previous boot environment
- Clean up old boot environments to free space
- Perform update via console instead of web UI
- Update incrementally instead of skipping versions

### Migration Issues

**Symptoms:**
- Problems after migrating from CORE to SCALE
- Service configuration lost after migration

**Diagnostic Steps:**
1. Check migration logs
2. Verify compatibility of hardware with new version
3. Review service configurations

**Solutions:**
- Restore from backup if migration fails badly
- Reconfigure services manually if needed
- Check community forums for GMKTEC-specific migration issues
- Consider clean installation rather than migration for major version changes

## Diagnostic Procedures and Log Analysis

### System Logs

#### Key Log Files to Check

- `/var/log/messages`: General system messages
- `/var/log/debug.log`: Detailed debug information
- `/var/log/samba4/log.smbd`: SMB service logs
- `/var/log/middlewared.log`: TrueNAS middleware logs

**How to Access Logs:**
1. Via Web Interface:
   - Navigate to System > Advanced > View Logs
   
2. Via Shell:
   ```
   cat /var/log/messages
   tail -f /var/log/debug.log
   ```

#### Common Log Messages and Meaning

| Log Message | Meaning | Action |
|-------------|---------|--------|
| `kernel: ACPI Error` | BIOS/ACPI issue | Update BIOS or add kernel parameters |
| `ZFS: unable to join pool` | Pool import issue | Check drive connections, try manual import |
| `Failed to connect: No route to host` | Network routing problem | Check network settings and connectivity |
| `Memory allocation failure` | Out of memory condition | Check memory usage, increase swap or RAM |

### Diagnostic Commands

#### Network Diagnostics

```bash
# Check network interfaces
ifconfig

# Test connectivity
ping gateway_ip
traceroute external_site

# Check network services
netstat -tuln

# Test DNS resolution
nslookup example.com
```

#### Storage Diagnostics

```bash
# Check ZFS pool status
zpool status
zpool list

# Check dataset information
zfs list
zfs get all pool/dataset

# Check drive health
smartctl -a /dev/sdX
```

#### Performance Diagnostics

```bash
# Check system load
uptime

# Monitor system resources
top
iostat -xm 2

# ZFS cache statistics
arc_summary
```

## Recovery Procedures

### Boot Environment Recovery

**When to Use:**
- System won't boot after update
- Configuration change caused system instability

**Steps:**
1. Reboot the system
2. At the boot menu, select a previous boot environment
3. Once booted, navigate to System > Boot to set default

### Pool Import Recovery

**When to Use:**
- Storage pool not automatically imported
- Moved drives from another system

**Steps:**
1. Navigate to Storage > Import Pool
2. If not visible, try via Shell:
   ```
   zpool import -f poolname
   ```
3. For pools from another system:
   ```
   zpool import -f oldpoolname newpoolname
   ```

### Configuration Backup and Restore

**Backup Procedure:**
1. Navigate to System > General > Save Config
2. Store the configuration file safely

**Restore Procedure:**
1. Navigate to System > General > Upload Config
2. Select saved configuration file
3. Apply and reboot as needed

### Emergency File Recovery

**When to Use:**
- Accidentally deleted critical files
- Need to restore from a snapshot

**Steps:**
1. Navigate to dataset containing snapshots
2. Click on the desired snapshot
3. Browse to the file you need to recover
4. Use "Restore" or copy to a new location

### Full System Recovery

**When to Use:**
- Catastrophic system failure
- Boot device failure

**Steps:**
1. Install TrueNAS on a new boot device
2. Import existing pools
3. Restore configuration from backup
4. Reconfigure services if needed

## GMKTEC-Specific Recovery Notes

### BIOS Recovery

**When to Use:**
- BIOS settings corrupted
- System won't POST

**Steps:**
1. Power off the system completely
2. Remove power cable
3. Wait 10 seconds
4. Hold reset/recovery button if available (check GMKTEC model manual)
5. Reconnect power and boot

### Hardware Reset

If your GMKT

