# HP ProLiant DL360 G6 Configuration Guide

This repository provides a comprehensive guide for configuring an HP ProLiant DL360 G6 server, which is anything but trivial. This guide covers the complete setup process with Ubuntu Server 24.04.5 LTS, including firmware updates, licensing, and Docker installation.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Configuration Steps](#configuration-steps)
  - [1. Update iLO2 to Version 2.33](#1-update-ilo2-to-version-233)
  - [2. Obtain iLO Advanced License](#2-obtain-ilo-advanced-license)
  - [3. Update BIOS and Dependencies with HP SPP](#3-update-bios-and-dependencies-with-hp-spp)
  - [4. Install Ubuntu Server 24.04.5 LTS](#4-install-ubuntu-server-24045-lts)
  - [5. Install Docker](#5-install-docker)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## Overview

The HP ProLiant DL360 G6 is a legacy server that requires specific configuration steps to work with modern operating systems. This guide walks you through:

- Updating the Integrated Lights-Out 2 (iLO2) firmware to the latest version (2.33)
- Acquiring and installing the required iLO Advanced license for virtual media functionality
- Updating BIOS and system dependencies using HP Service Pack for ProLiant (SPP)
- Installing Ubuntu Server 24.04.5 LTS via virtual drive
- Setting up Docker for containerized applications

## Prerequisites

Before starting, ensure you have:

- HP ProLiant DL360 G6 server with network connectivity
- Access to the iLO2 web interface
- Approximately 11€ for the iLO Advanced license
- Internet connection for downloading required files
- Basic understanding of server administration

### Required Downloads

1. **iLO2 Firmware 2.33**: Latest iLO2 firmware
2. **HP SPP2017040.2017 ISO**: Service Pack for ProLiant (updates BIOS and dependencies)
3. **Ubuntu Server 24.04.5 LTS ISO**: Operating system installation media
4. **iLO Advanced License**: Purchase from HP or authorized reseller

## Configuration Steps

### 1. Update iLO2 to Version 2.33

The Integrated Lights-Out (iLO) is HP's remote management solution. Updating to the latest version ensures compatibility and security.

#### Steps:

1. **Access iLO2 Web Interface**
   - Connect to your iLO2 IP address via web browser
   - Default credentials are often on a pull-out tag on the server
   - URL format: `https://<ilo-ip-address>`

2. **Check Current iLO Version**
   - Log into iLO web interface
   - Navigate to **Information** → **Overview**
   - Note the current firmware version

3. **Download iLO2 Firmware 2.33**
   - Visit HP Support website
   - Search for "iLO2 firmware 2.33"
   - Download the firmware file (usually a `.bin` file)

4. **Update iLO Firmware**
   - In iLO web interface, go to **Administration** → **Firmware**
   - Click **Browse** and select the downloaded firmware file
   - Click **Upload Firmware**
   - Wait for the upload and installation to complete (do not interrupt)
   - iLO will automatically reboot after the update
   - Reconnect and verify the new version (2.33)

### 2. Obtain iLO Advanced License

The iLO Advanced license is required for virtual media functionality, which allows you to mount ISO images remotely.

#### Why You Need It:

- Virtual Media support (mount ISO files remotely)
- Remote console capabilities
- Virtual power and reset control
- Essential for installing operating systems without physical media

#### How to Obtain:

1. **Purchase the License**
   - Cost: Approximately 11€
   - Available from HP or authorized resellers
   - Search for "HP iLO Advanced License" or "iLO2 Advanced License"
   - You will receive a license key after purchase

2. **Install the License**
   - Log into iLO2 web interface
   - Navigate to **Administration** → **Licensing**
   - Enter the license key in the appropriate field
   - Click **Activate**
   - Verify that "Advanced" features are now available

### 3. Update BIOS and Dependencies with HP SPP

The HP Service Pack for ProLiant (SPP) is a comprehensive collection of firmware and system software updates.

#### Using SPP2017040.2017 ISO:

1. **Download HP SPP2017040.2017**
   - Download the SPP ISO from HP Support website
   - File size is typically several GB
   - Verify the ISO checksum after download

2. **Mount SPP ISO via iLO Virtual Media**
   - Log into iLO2 web interface
   - Navigate to **Remote Console** → **Virtual Media**
   - Click **CD/DVD-ROM Image** → **Browse**
   - Select the downloaded SPP ISO file
   - Click **Insert Media**

3. **Boot from Virtual CD/DVD**
   - In iLO, go to **Remote Console** → **Launch Console**
   - Power on or reset the server
   - Press **F11** during POST to enter Boot Menu
   - Select the virtual CD/DVD drive
   - The SPP interface will load

4. **Update Firmware and BIOS**
   - In the SPP interface, select **Guided Update**
   - Review available updates (BIOS, system firmware, etc.)
   - Select all recommended updates
   - Begin the update process
   - **Important**: Do not power off or interrupt during updates
   - Server will reboot multiple times during the process
   - Total time can be 30-60 minutes depending on updates

5. **Verify Updates**
   - After completion, boot into system setup (F9 during POST)
   - Verify BIOS version has been updated
   - Check iLO for updated firmware versions

### 4. Install Ubuntu Server 24.04.5 LTS

With updated firmware and iLO Advanced license, you can now install Ubuntu Server via virtual media.

#### Steps:

1. **Download Ubuntu Server 24.04.5 LTS ISO**
   - Visit Ubuntu's official website
   - Download the 64-bit ISO for Ubuntu Server 24.04.5 LTS
   - Verify the ISO checksum

2. **Mount Ubuntu ISO via iLO Virtual Media**
   - Log into iLO2 web interface
   - Navigate to **Remote Console** → **Virtual Media**
   - Eject any previously mounted ISO
   - Click **CD/DVD-ROM Image** → **Browse**
   - Select the Ubuntu Server ISO
   - Click **Insert Media**

3. **Boot from Virtual Media**
   - In iLO Remote Console, power on or reset the server
   - Press **F11** to enter Boot Menu
   - Select the virtual CD/DVD drive
   - Ubuntu installer will load

4. **Install Ubuntu Server**
   - Follow the Ubuntu installation wizard
   - Select language and keyboard layout
   - Configure network settings
   - Set up disk partitioning (recommended: use LVM)
   - Create user account
   - Select additional software (OpenSSH server recommended)
   - Complete installation
   - Remove virtual media and reboot

5. **Post-Installation**
   - Log in to the server
   - Update system packages:
     ```bash
     sudo apt update
     sudo apt upgrade -y
     ```
   - Configure additional settings as needed

### 5. Install Docker

Docker is required for containerized applications, including Javafox.

#### Installation Steps:

1. **Install Docker Using Official Repository**
   ```bash
   # Update package index
   sudo apt update
   
   # Install prerequisites
   sudo apt install -y ca-certificates curl gnupg lsb-release
   
   # Add Docker's official GPG key
   sudo install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   sudo chmod a+r /etc/apt/keyrings/docker.gpg
   
   # Set up Docker repository
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   
   # Install Docker Engine
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

2. **Verify Docker Installation**
   ```bash
   sudo docker run hello-world
   ```

3. **Add User to Docker Group** (optional, for non-root access)
   ```bash
   sudo usermod -aG docker $USER
   newgrp docker
   ```

4. **Install Docker Compose** (if not already included)
   ```bash
   # Docker Compose is typically included with Docker Engine
   docker compose version
   ```

5. **Configure Docker for Javafox**
   - Javafox is a containerized application
   - Follow Javafox-specific documentation for deployment
   - Typically involves running:
     ```bash
     docker pull <javafox-image>
     docker run -d --name javafox <javafox-image>
     ```

## Troubleshooting

### iLO Issues

**Problem**: Cannot access iLO web interface
- **Solution**: Check network cable connections, verify iLO has an IP address, ensure you're on the same network or have routing configured

**Problem**: iLO firmware update fails
- **Solution**: Ensure firmware file is not corrupted, try a different browser, verify you have administrator privileges

### SPP Update Issues

**Problem**: Server hangs during firmware update
- **Solution**: Wait at least 60 minutes before assuming failure. If truly stuck, power cycle and try individual component updates instead of guided update

**Problem**: Cannot boot from virtual media
- **Solution**: Verify iLO Advanced license is active, ensure ISO is properly mounted, check boot order in BIOS

### Ubuntu Installation Issues

**Problem**: Network not detected during installation
- **Solution**: Check physical network connections, try different network port, may need to configure network manually

**Problem**: Installer cannot find hard drives
- **Solution**: Check RAID configuration in Smart Array controller (F8 during POST), ensure drives are properly configured in a RAID array

### Docker Issues

**Problem**: Permission denied when running Docker commands
- **Solution**: Either run with `sudo` or add your user to the docker group: `sudo usermod -aG docker $USER`

**Problem**: Docker service won't start
- **Solution**: Check system logs: `sudo journalctl -u docker`, ensure virtualization is enabled in BIOS if using VMs

## Resources

### Official Documentation

- [HP iLO 2 User Guide](https://support.hpe.com/hpesc/public/home)
- [HP Service Pack for ProLiant](https://support.hpe.com/hpesc/public/home)
- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [Docker Documentation](https://docs.docker.com/)

### Downloads

- [HP Support and Drivers](https://support.hpe.com/hpesc/public/home)
- [Ubuntu Server Downloads](https://ubuntu.com/download/server)
- [Docker Engine Installation](https://docs.docker.com/engine/install/ubuntu/)

### Community Resources

- [HP ProLiant Community Forums](https://community.hpe.com/)
- [Ubuntu Forums](https://ubuntuforums.org/)
- [Docker Community Forums](https://forums.docker.com/)

## Notes

- The HP ProLiant DL360 G6 is a legacy server from 2009-2010 era
- While it can run modern operating systems, performance will be limited compared to current hardware
- Consider power consumption and noise levels when deploying
- Regular backups are essential for any server deployment
- Keep firmware and software updated for security

## License

This documentation is provided under the MIT License. See [LICENSE](LICENSE) for details.

## Contributing

Contributions to improve this guide are welcome! Please submit issues or pull requests with your suggestions.

---

*Last updated: January 2026*