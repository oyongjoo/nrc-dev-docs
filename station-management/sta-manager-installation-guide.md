# STA Manager Installation Guide

## Overview
This guide covers the installation and configuration of the STA Manager system for OpenWrt HaLow networks.

## Prerequisites
- OpenWrt HaLow firmware installed
- SSH access to the device
- Basic understanding of OpenWrt package management

## Installation Steps

### 1. Package Installation
```bash
# Install the STA Manager package
opkg update
opkg install sta-manager
```

### 2. Service Configuration
```bash
# Enable the service
/etc/init.d/sta_manager enable

# Start the service
/etc/init.d/sta_manager start
```

### 3. Database Initialization
The STA Manager will automatically create the required SQLite database on first run.

### 4. Verification
```bash
# Check service status
/etc/init.d/sta_manager status

# View connected stations
sta_query -c
```

## Configuration Options

Edit `/etc/config/sta_manager` to customize:
- Database location
- Logging level
- Update intervals

## Troubleshooting

### Service Won't Start
- Check log files: `/var/log/sta_manager.log`
- Verify database permissions
- Ensure SQLite is installed

### Database Issues
- Reset database: `rm /var/lib/sta_manager.db`
- Restart service to recreate

---
*Last updated: 2025-07-29*