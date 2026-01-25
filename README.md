# HP ProLiant DL360 G6 Configuration Guide

This repository provides a comprehensive guide for configuring an HP ProLiant DL360 G6 server, which is anything but trivial. This guide covers the complete setup process with Ubuntu Server 24.04.5 LTS, including firmware updates, licensing, and the required Docker setup for accessing the legacy iLO2 interface.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Configuration Steps](#configuration-steps)
  - [1. Setup Javafox Docker Container (Required for iLO2 Access)](#1-setup-javafox-docker-container-required-for-ilo2-access)
  - [2. Update iLO2 to Version 2.33](#2-update-ilo2-to-version-233)
  - [3. Obtain iLO Advanced License](#3-obtain-ilo-advanced-license)
  - [4. Update BIOS and Dependencies with HP SPP](#4-update-bios-and-dependencies-with-hp-spp)
  - [5. Install Ubuntu Server 24.04.5 LTS](#5-install-ubuntu-server-24045-lts)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## Overview

The HP ProLiant DL360 G6 is a legacy server that requires specific configuration steps to work with modern operating systems. This guide walks you through:

- **Setting up Javafox Docker container** - Required to access the iLO2 web interface (modern browsers don't support the legacy iLO2)
- Updating the Integrated Lights-Out 2 (iLO2) firmware to the latest version (2.33)
- Acquiring and installing the required iLO Advanced license for virtual media functionality
- Updating BIOS and system dependencies using HP Service Pack for ProLiant (SPP)
- Installing Ubuntu Server 24.04.5 LTS via virtual drive

## Prerequisites

Before starting, ensure you have:

- HP ProLiant DL360 G6 server with network connectivity
- A computer with Docker installed (required to run Javafox for iLO2 access)
- Network access to the iLO2 IP address from your computer
- Approximately 11€ for the iLO Advanced license (price may vary by region and vendor)
- Internet connection for downloading required files
- Basic understanding of server administration and Docker

**Important**: Modern browsers (Chrome, Firefox, Edge) do not support the legacy Java applets and ActiveX controls required by iLO2. You **must** use the Javafox Docker container to access the iLO2 web interface and virtual media functionality.

### Required Downloads

1. **iLO2 Firmware 2.33**: Latest iLO2 firmware
2. **HP SPP2017040.2017 ISO**: Service Pack for ProLiant (updates BIOS and dependencies)
3. **Ubuntu Server 24.04.5 LTS ISO**: Operating system installation media
4. **iLO Advanced License**: Purchase from HP or authorized reseller

## Configuration Steps

### 1. Setup Javafox Docker Container (Required for iLO2 Access)

**Critical First Step**: The iLO2 web interface requires legacy Java applets and browser plugins that are no longer supported by modern browsers. The Javafox Docker container provides an old Firefox browser environment that can properly access iLO2's web interface and virtual media features.

#### Why Javafox is Required:

- iLO2 was designed in the late 2000s and relies on Java applets and legacy browser technologies
- Modern browsers (Chrome, Firefox, Edge, Safari) have removed support for these technologies for security reasons
- Without Javafox, you **cannot** access iLO2's virtual media, remote console, or many administrative features
- Javafox provides a containerized old Firefox browser that still supports these legacy technologies

#### Installation Steps:

1. **Install Docker on Your Local Computer** (if not already installed)
   
   On Ubuntu/Debian:
   ```bash
   # Update package index
   sudo apt update
   
   # Install Docker
   sudo apt install -y docker.io
   
   # Start Docker service
   sudo systemctl start docker
   sudo systemctl enable docker
   
   # Add your user to docker group (to run without sudo)
   sudo usermod -aG docker $USER
   newgrp docker
   ```
   
   On macOS:
   - Download and install Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)
   
   On Windows:
   - Download and install Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)

2. **Pull and Run the Javafox Docker Container**
   ```bash
   # Pull the Javafox image (old Firefox with Java support)
   docker pull jlesage/firefox:v1.17.0
   
   # Run Javafox container with web interface on port 5800
   docker run -d \
     --name javafox \
     -p 5800:5800 \
     -v /tmp/javafox:/config:rw \
     jlesage/firefox:v1.17.0
   ```

3. **Access Javafox**
   - Open your modern browser and go to: `http://localhost:5800`
   - You will see a Firefox browser running in your browser window
   - This Firefox instance supports the legacy Java applets needed for iLO2

4. **Access iLO2 Through Javafox**
   - In the Javafox browser window, navigate to your iLO2 IP address: `https://<ilo-ip-address>`
   - Accept the security certificate warning (iLO2 uses self-signed certificates)
   - You can now log in and use all iLO2 features including virtual media
   - **Keep this Javafox container running** throughout the configuration process

5. **Alternative: Use Javafox on a Different Machine**
   - If your main computer can't run Docker, install Javafox on another machine
   - Make sure that machine has network access to the iLO2 IP address
   - Access Javafox via `http://<docker-host-ip>:5800` from any browser

**Note**: All subsequent steps that require accessing the iLO2 web interface must be done through the Javafox browser.

### 2. Update iLO2 to Version 2.33

The Integrated Lights-Out (iLO) is HP's remote management solution. Updating to the latest version ensures compatibility and security.

#### Steps:

1. **Access iLO2 Web Interface via Javafox**
   - Open Javafox in your browser: `http://localhost:5800`
   - In the Javafox Firefox browser, navigate to: `https://<ilo-ip-address>`
   - Default credentials are often on a pull-out tag on the server
   - Accept the self-signed certificate warning

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

### 3. Obtain iLO Advanced License

The iLO Advanced license is required for virtual media functionality, which allows you to mount ISO images remotely.

#### Why You Need It:

- Virtual Media support (mount ISO files remotely)
- Remote console capabilities
- Virtual power and reset control
- Essential for installing operating systems without physical media

#### How to Obtain:

1. **Purchase the License**
   - Cost: Approximately 11€ (price may vary by region, vendor, and current promotions)
   - Available from HP or authorized resellers
   - Search for "HP iLO Advanced License" or "iLO2 Advanced License"
   - You will receive a license key after purchase
   - **Note**: Check if your organization has existing licenses or enterprise agreements before purchasing

2. **Install the License**
   - Log into iLO2 web interface
   - Navigate to **Administration** → **Licensing**
   - Enter the license key in the appropriate field
   - Click **Activate**
   - Verify that "Advanced" features are now available

### 4. Update BIOS and Dependencies with HP SPP

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

### 5. Install Ubuntu Server 24.04.5 LTS

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

## Troubleshooting

### Javafox / Docker Issues

**Problem**: Cannot access Javafox at localhost:5800
- **Solution**: Ensure Docker container is running: `docker ps | grep javafox`. If not running, restart it: `docker start javafox`

**Problem**: Javafox shows but iLO2 doesn't load
- **Solution**: Check network connectivity from Docker container to iLO2 IP. Verify iLO2 IP address is correct and iLO2 is powered on.

**Problem**: Java applets don't work in Javafox
- **Solution**: The jlesage/firefox image should have Java support. If issues persist, try an alternative image like `jlesage/firefox:v1.17.0` or earlier versions.

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