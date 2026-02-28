# Setting up a Samba File Share on Ubuntu Server
**Author:** Greg Diny  
**Date:** November 15, 2025

A practical guide to setting up a local network file share using Samba on Ubuntu Server.

## Overview

This guide covers setting up a Samba file share on Ubuntu Server for local network file access. Samba implements the SMB (Server Message Block) protocol, which is what Windows uses for file sharing. This allows seamless file access from Windows, Mac, and Linux clients.

**Hardware Used:** Dell Wyse 5070 (Intel Celeron J4105, 8GB RAM, 256GB SSD)

## Prerequisites

- Fresh Ubuntu Server install
- User account created (e.g., `file_manager`)
- Static IP configured (e.g., `192.168.30.20`)
- Network access to the server

## Security Note

⚠️ **SMB uses port 445**, one of the most scanned ports on the internet. **Never expose this port to the internet.** Always include firewall rules to block port 445 from WAN access.

## Installation Steps

### 1. Update System

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Samba

```bash
sudo apt install -y samba
```

Verify it's running:
```bash
systemctl status smbd --no-pager
```

You should see `active (running)`.

### 3. Create Shared Directory

```bash
# Create the share directory
sudo mkdir -p /srv/samba/shared

# Set ownership to your user
sudo chown -R file_manager:file_manager /srv/samba/shared

# Set permissions (2775 ensures files inherit group)
sudo chmod -R 2775 /srv/samba/shared
```

### 4. Add Samba User

Samba uses its own password database, separate from Linux login:

```bash
# Add user to Samba
sudo smbpasswd -a file_manager

# Enable the user
sudo smbpasswd -e file_manager
```

You'll be prompted to set the SMB password.

### 5. Configure Samba

Backup the config first:
```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

Edit the config:
```bash
sudo nano /etc/samba/smb.conf
```

Add this to the bottom:
```ini
[shared]
    path = /srv/samba/shared
    browseable = yes
    writable = yes
    valid users = file_manager
    create mask = 0664
    directory mask = 2775
    force user = file_manager
    force group = file_manager
```

Save and exit (`Ctrl+X`, `Y`, `Enter`).

### 6. Test Configuration

```bash
testparm
```

This validates your Samba config. Fix any errors before continuing.

### 7. Restart Samba

```bash
sudo systemctl restart smbd
sudo systemctl enable smbd
```

### 8. Configure Firewall (UFW)

```bash
# Allow Samba on local network only
sudo ufw allow from 192.168.30.0/24 to any port 445
sudo ufw allow from 192.168.30.0/24 to any port 139

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

## Connecting from Clients

### Windows

1. Open File Explorer
2. In address bar: `\\192.168.30.20\shared`
3. Enter credentials:
   - Username: `file_manager`
   - Password: (your Samba password)

### macOS

1. Finder → Go → Connect to Server
2. Server address: `smb://192.168.30.20/shared`
3. Enter credentials when prompted

### Linux

```bash
# Install cifs-utils
sudo apt install cifs-utils

# Create mount point
sudo mkdir -p /mnt/samba

# Mount manually
sudo mount -t cifs //192.168.30.20/shared /mnt/samba -o username=file_manager

# Or add to /etc/fstab for auto-mount
```

## Troubleshooting

**Can't connect from Windows:**
- Verify firewall allows SMB ports
- Check Windows network discovery is enabled
- Ensure you're on the same network/VLAN

**Permission denied:**
- Verify user exists in Samba: `sudo pdbedit -L`
- Check directory permissions: `ls -la /srv/samba/shared`
- Confirm correct username/password

**Service won't start:**
- Check config syntax: `testparm`
- Review logs: `sudo journalctl -u smbd -n 50`
- Verify port 445 isn't already in use: `sudo netstat -tulpn | grep 445`

## Advanced: Web Access with Apache

For web-based file access, you can install Apache alongside Samba. Apache provides HTTP access while Samba provides SMB access.

```bash
sudo apt install apache2
sudo a2enmod autoindex
# Configure DocumentRoot to point to /srv/samba/shared
```

## Security Best Practices

1. ✅ Never expose port 445 to the internet
2. ✅ Use strong Samba passwords
3. ✅ Restrict access with `valid users` directive
4. ✅ Use firewall rules to limit access to local network
5. ✅ Consider VPN for remote access instead of port forwarding
6. ✅ Regular updates: `sudo apt update && sudo apt upgrade`

---

*This guide was developed through hands-on implementation on a Dell Wyse 5070 running Ubuntu Server in a home lab environment.*
