# TrueNAS Configuration Guide for GMKTEC Homelab

This guide covers the post-installation configuration of TrueNAS on GMKTEC hardware, focusing on practical homelab usage. Follow these steps to set up your TrueNAS system for optimal performance, security, and usability.

## Initial System Settings and Security Hardening

After completing the [installation process](installation-guide.md), follow these steps to configure basic system settings and improve security.

### 1. System Settings

1. **General Settings**
   - Navigate to System > General
   - Set the appropriate timezone
   - Configure GUI settings according to your preference
   - Enable HTTPS for web interface access

2. **Update System**
   - Navigate to System > Update
   - Check for and install any available updates
   - Enable automatic updates if desired

3. **Network Configuration**
   - Navigate to Network > Global Configuration
   - Set a descriptive hostname (e.g., `truenas-gmktec`)
   - Configure DNS servers (consider using reliable DNS like Cloudflare's `1.1.1.1`)
   - Navigate to Network > Interfaces
   - Configure the primary network interface with a static IP address

4. **Boot Environment Management**
   - Navigate to System > Boot
   - Keep at least 2-3 known working boot environments
   - Set up automatic boot environment creation before updates

### 2. Security Hardening

1. **Change Default Passwords**
   - Navigate to Accounts > Users
   - Select the root user and set a strong password
   - Consider creating a separate admin account for day-to-day management

2. **SSH Configuration**
   - Navigate to Services > SSH
   - Disable root login via SSH
   - Change the default SSH port (optional but recommended)
   - Configure SSH key authentication instead of password authentication
   - Set appropriate timeout values

3. **Firewall Setup**
   - Navigate to Network > Firewall
   - Enable the firewall
   - Create rules to only allow necessary traffic:
     - Allow HTTP/HTTPS (ports 80/443) for web interface
     - Allow SSH on your custom port
     - Allow required ports for file sharing services you plan to use
     - Restrict access to specific IP ranges when possible

4. **Two-Factor Authentication**
   - Navigate to System > Two-Factor Authentication
   - Enable and configure 2FA for additional security
   - Store backup codes in a secure location

5. **Certificate Management**
   - Navigate to System > Certificates
   - Create or import a valid SSL certificate
   - Configure the web interface to use HTTPS with your certificate

## Storage Pool and Dataset Configuration

Proper storage configuration is critical for data integrity, performance, and organization.

### 1. Storage Pool Creation

1. **Plan Your Storage Layout**
   - Consider your GMKTEC hardware limitations:
     - Number of available SATA ports
     - Drive bay limitations
     - External storage options

2. **Create Storage Pool**
   - Navigate to Storage > Pools > Add
   - Select Manual for advanced configuration
   - Choose appropriate RAID level:
     - Mirror (RAID1): For 2 drives, best protection
     - RAIDZ1: Minimum 3 drives, one drive failure tolerance
     - RAIDZ2: Minimum 4 drives, two drive failure tolerance
   - Name your pool (e.g., "tank", "data", "storage")
   - Set checksum algorithm to Fletcher4 (default, good balance)
   - For GMKTEC mini PCs with limited internal storage:
     - Consider using external USB drives in mirror configuration
     - Use the fastest available connection (USB 3.1/3.2 preferred)

3. **Optimize Pool Settings**
   - Navigate to your new pool's settings
   - Enable automatic scrubs (recommended monthly)
   - Configure auto-trim if using SSDs

### 2. Dataset Organization

1. **Plan Your Dataset Hierarchy**
   - Create a logical structure based on your needs
   - Common organization patterns:
     - By content type: Media, Documents, Backups
     - By user: UserA, UserB
     - By function: Shared, Private, Backups

2. **Create Datasets**
   - Navigate to Storage > Pools > [Your Pool] > Add Dataset
   - Name each dataset according to your plan
   - Configure appropriate settings for each:
     - Compression: LZ4 (good balance of performance/compression)
     - Share Type: SMB or NFS depending on intended usage
     - Quota: Set limits to prevent any single dataset from filling the pool

3. **Example Dataset Structure**
   ```
   tank/
   ├── media/
   │   ├── movies
   │   ├── music
   │   └── photos
   ├── documents/
   │   ├── business
   │   └── personal
   ├── backups/
   │   ├── computers
   │   └── phones
   └── shared/
       ├── family
       └── projects
   ```

### 3. Special Datasets and Considerations

1. **iSCSI Volumes (if needed)**
   - Navigate to Sharing > Block Shares (iSCSI)
   - Create zvols with appropriate size and block size
   - Configure targets and extents
   - Use for virtualization storage or dedicated application needs

2. **Jails/VMs/Apps Storage (TrueNAS SCALE)**
   - Create dedicated datasets for applications
   - Consider performance needs when placing these datasets

## User and Group Management

Proper user management ensures secure and organized access to your data.

### 1. User Management

1. **Create User Groups**
   - Navigate to Accounts > Groups > Add
   - Create groups based on access needs (e.g., family, media, admin)

2. **Create Users**
   - Navigate to Accounts > Users > Add
   - Create individual accounts for each user
   - Set strong passwords or configure SSH key authentication
   - Assign users to appropriate groups
   - Set home directories if users will store personal files

3. **Special Considerations**
   - Create service accounts for specific applications with limited permissions
   - Consider creating a dedicated backup user with limited scope

### 2. Permissions Management

1. **Set Appropriate Permissions**
   - Navigate to your datasets
   - Set ownership and permissions:
     - Owner: appropriate user or group
     - Group: appropriate group
     - Permissions: based on needed access levels

2. **ACL Configuration**
   - For more granular control, use ACLs
   - Navigate to dataset > Edit Permissions
   - Configure ACLs based on your sharing needs

3. **Inheritance Settings**
   - Configure permission inheritance for child datasets
   - This simplifies management of nested directories

## Share Configuration (SMB, NFS)

Configure file sharing to make your data accessible to clients.

### 1. SMB Shares (Windows File Sharing)

1. **Enable SMB Service**
   - Navigate to Services > SMB
   - Enable the service and set to start automatically
   - Configure basic settings:
     - Workgroup: Match your Windows network
     - NetBIOS Name: Your server name
     - Enable logging at an appropriate level

2. **Create SMB Shares**
   - Navigate to Sharing > Windows Shares (SMB)
   - Add a new share:
     - Path: Select the dataset to share
     - Name: Give it a meaningful name
     - Access Control:
       - Allow guest access or
       - Require user authentication (recommended)
     - Permissions: Configure appropriately

3. **Advanced SMB Configuration**
   - Configure shadow copies for Windows Previous Versions
   - Set appropriate performance tuning:
     - For GMKTEC hardware with limited RAM, set reasonable max connections
     - Adjust read/write buffer sizes based on available memory

### 2. NFS Shares (Unix/Linux File Sharing)

1. **Enable NFS Service**
   - Navigate to Services > NFS
   - Enable the service and set to start automatically
   - Configure appropriate settings:
     - Number of servers: Set based on expected load (start with 4)
     - Bind IP addresses: Limit to specific network interfaces if needed

2. **Create NFS Shares**
   - Navigate to Sharing > Unix Shares (NFS)
   - Add a new share:
     - Path: Select the dataset to share
     - Authorized networks: Limit to your local subnet (e.g., 192.168.1.0/24)
     - Mapall User/Group: Map permissions appropriately

3. **NFS Security Configuration**
   - Configure appropriate security options based on your needs
   - For homelab use, limiting to authorized networks is often sufficient

### 3. Other Sharing Options

1. **FTP/SFTP (if needed)**
   - Configure only if specifically required
   - Prefer SFTP over FTP for security
   - Limit to specific users and directories

2. **WebDAV (if needed)**
   - Enable for web-based access to files
   - Always configure with HTTPS

## Backup Strategies

Implement a comprehensive backup strategy to protect your data.

### 1. Internal Backup Strategies

1. **ZFS Snapshots**
   - Navigate to Data Protection > Snapshots
   - Configure periodic snapshots:
     - Hourly: Keep 24
     - Daily: Keep 7-30
     - Weekly: Keep 4-12
     - Monthly: Keep 3-12
   - Configure snapshot retention policies based on dataset importance

2. **Replication**
   - If you have multiple storage pools:
     - Navigate to Data Protection > Replication Tasks
     - Configure replication between pools for critical data

### 2. External Backup Strategies

1. **Cloud Backup**
   - Install rclone or similar tool via:
     - TrueNAS CORE: Plugins or Jail
     - TrueNAS SCALE: Apps
   - Configure automated backups to cloud services:
     - Backblaze B2
     - Amazon S3
     - Google Drive
     - OneDrive

2. **Local External Backup**
   - Connect an external USB drive to your GMKTEC system
   - Create a periodic replication task to this drive
   - Consider rotating multiple external drives

3. **Remote TrueNAS Backup**
   - If you have another TrueNAS system, configure replication:
     - Navigate to Data Protection > Replication Tasks
     - Set up synchronization between systems

### 3. Backup Verification

1. **Regular Testing**
   - Periodically test restoration from:
     - Snapshots
     - External backups
     - Cloud backups
   - Document the verification process and results

## Monitoring and Notification Setup

Configure monitoring to stay informed about your system's health.

### 1. Alert Services

1. **Email Notifications**
   - Navigate to System > Alert Services
   - Configure email settings:
     - SMTP server
     - Authentication credentials
     - Recipients for alerts

2. **Other Notification Methods**
   - Configure additional alert methods:
     - Slack
     - Telegram
     - PushOver
     - SNMP Traps

### 2. Alert Settings

1. **Configure Alert Rules**
   - Navigate to System > Alert Settings
   - Adjust alert levels based on importance:
     - Critical: Immediate attention required
     - Warning: Issues to address soon
     - Notice: Informational alerts

2. **Custom Alert Conditions**
   - Create custom alert rules for specific conditions
   - Focus on metrics important for your environment

### 3. Reporting

1. **Periodic Reports**
   - Navigate to System > Reporting
   - Configure regular system reports
   - Set appropriate metrics to monitor

2. **External Monitoring Tools**
   - Consider adding:
     - Grafana for visualization
     - Prometheus for metrics collection
     - LibreNMS for network monitoring

### 4. SMART Monitoring

1. **Drive Health Monitoring**
   - Navigate to Tasks > S.M.A.R.T. Tests
   - Configure periodic tests:
     - Short tests: Weekly
     - Long tests: Monthly
   - Configure email alerts for drive issues

## Performance Tuning

Optimize your TrueNAS system for the GMKTEC hardware constraints.

### 1. Memory Optimization

1. **ZFS Arc Cache Tuning**
   - For GMKTEC systems with limited RAM (16GB or less):
     - Navigate to System > Advanced
     - Set ARC maximum to 50% of available RAM
   - For systems with more RAM:
     - Allow up to 75% for ARC cache

2. **Swap Configuration**
   - Configure appropriate swap size based on RAM
   - For low-memory systems, ensure adequate swap space

### 2. Network Tuning

1. **Jumbo Frames**
   - If your network supports it:
     - Navigate to Network > Interfaces
     - Set MTU to 9000 on all interfaces
     - Ensure all network equipment supports jumbo frames

2. **Link Aggregation**
   - If your GMKTEC system has multiple NICs:
     - Navigate to Network > Interface
     - Create a Link Aggregation
     - Choose appropriate aggregation protocol (LACP recommended)

### 3. Storage Performance

1. **SSD Integration**
   - For hybrid storage configurations:
     - Use SSDs for L2ARC (read cache) if you have RAM > 32GB
     - Use SSDs for SLOG (write cache) for sync-heavy workloads

2. **Dataset Optimization**
   - Tune dataset properties:
     - Record size: Adjust based on expected file sizes
     - Special Small Block Size: For datasets with many small files
     - Sync settings: Based on data importance vs. performance needs

### 4. Background Services

1. **Service Priority**
   - Adjust service priorities:
     - Ensure critical services get priority
     - Lower priority for background tasks

2. **Scheduled Tasks**
   - Schedule intensive tasks during off-peak hours:
     - Scrubs
     - SMART tests
     - Snapshots
     - Cloud backups

### 5. GMKTEC-Specific Optimizations

1. **Thermal Management**
   - Monitor system temperatures
   - Ensure adequate cooling for your mini PC
   - Consider setting up temperature-based fan control if supported

2. **Power Management**
   - Configure HDD spin-down settings to reduce power consumption
   - Balance between power savings and access latency

## Example Configurations for Common Homelab Uses

### 1. Media Server Configuration

```
# Storage Layout
tank/
└── media/
    ├── movies     # SMB share for movies
    ├── tv         # SMB share for TV shows
    ├── music      # SMB share for music
    └── photos     # SMB share for photos
    
# Applications (TrueNAS SCALE)
- Plex Media Server
- Tdarr (for transcoding)
- Jellyfin (alternative media server)
```

### 2. Home Backup Server

```
# Storage Layout
tank/
└── backups/
    ├── computers/
    │   ├── laptop1
    │   ├── laptop2
    │   └── desktop
    ├── phones/
    │   ├── phone1
    │   └── phone2
    └── documents
    
# Applications
- Urbackup (centralized backup)
- Syncthing (file synchronization)
- Duplicati (cloud backup)
```

### 3. Development Environment

```
# Storage Layout
tank/
└── development/
    ├── projects
    ├── repositories
    ├── virtual_machines
    └── containers
    
# Applications (TrueNAS SCALE)
- GitLab
- Visual Studio Code Server
- Docker/Kubernetes environments
```

## Maintenance Tasks and Schedule

Establish a regular maintenance schedule to keep your system healthy.

### 1. Daily Tasks
- Check system alerts
- Verify successful backups

### 2. Weekly Tasks
- Review logs for anomalies
- Check storage usage trends
- Verify snapshot rotation

### 3. Monthly Tasks
- Run ZFS scrub on all pools
- Run long SMART tests on all drives
- Review system performance
- Check for TrueNAS updates

### 4. Quarterly Tasks
- Test restore from backups
- Review and update user access
- Clean dust from GMKTEC hardware (if accessible)
- Review and update documentation

## Conclusion

This configuration guide has covered the essential aspects of setting up TrueNAS on your GMKTEC hardware for homelab use. By following these recommendations, you'll have a robust, secure, and efficient NAS system

