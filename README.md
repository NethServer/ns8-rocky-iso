This repository contains scripts and instructions for creating an ISO image of Rocky Linux modified for the installation of NethServer 8, which includes NethVoice. The goal of this repository is to provide an image that can be used for appliances, which will serve as the leader node of a cluster (ideally a single-node cluster).

# Pre-requisites

- Internet connection
- At least 20 GB of disk space
- Minimum 2 CPUs
- Minimum 2 GB of RAM

# Creating the ISO image

- Option 1: Create the ISO image yourself
  - Download the ISO image from the site: [Rocky Linux Download](https://rockylinux.org/download)
  - Build the container file with the following command:
     ```sh
     buildah build -t ns8-boxbuilder .
     ```
  - Create the container that modifies the Rocky Linux ISO with the following command:
     ```sh
     podman run --rm -it --privileged -v $(pwd):/root localhost/ns8-boxbuilder mkksiso -R "Rocky Linux" "NethServer 8 (Rocky Linux)" --cmdline "inst.ks=https://raw.githubusercontent.com/NethServer/ns8-rocky-iso/refs/heads/main/ks.cfg" <downloaded_image_name>.iso ns8.iso
     ```
     > **Note:** Replace `<downloaded_image_name>` with the name of the downloaded ISO image.

- Option 2: Download the pre-built ISO image
  - You can download the pre-built ISO image directly from [this link](https://github.com/NethServer/ns8-rocky-iso/releases).

# Create bootable USB

- List block devices to identify the USB drive:
   ```sh
   lsblk
   ```
- Write the ISO file to the USB drive:
   ```sh
   sudo dd if=./ns8.iso of=/dev/sdc bs=4M oflag=direct status=progress
   ```
  **Note:** Replace `/dev/sdc` with the correct USB drive identifier

# Installation Process

1. **Boot from USB:**
   - Insert the USB drive into the target machine.
   - Boot the machine and select the USB drive as the boot device.

2. **Automatic Installation:**
   - The system will automatically start the installation process using the kickstart configuration file (`ks.cfg`).
   - The installation will proceed without user interaction, setting up the system language, keyboard, timezone, network configuration, root password, disk partitioning, and package selection.

3. **Post-Installation Script:**
   - After the installation, a post-installation script will run to configure the network, enable SSH root login, and install NethServer 8.
   - The script will also create a new cluster, add an internal provider, and install NethVoice Proxy and NethVoice.

4. **Network Configuration:**
   - The script will clean up existing network configurations and set a static IP address for the first network interface (192.168.1.1).
   - To change the network configuration, access the shell with `root` user and use the `nmtui` tool:
     ```sh
     nmtui
     ```
   - Follow the prompts to edit the connection, set a new IP address, and save the changes.

5. **Final Steps:**
   - The script will disable `rc.local` after the first boot and shut down the system.
   - Default configuration details can be found in the [Default Configurations](#default-configurations) section.

# Verifying Installation

- After the installation, the machine will shut down automatically if the installation is successful.
- Check the installation log for details:
   ```sh
   cat /var/log/ns8-install.log
   ```

# Default Configurations

- The system is paritioned with LVM
- Timezone is set to Europe/Rome
- Keyboard is set to Italian
- System language is English
- SSH access is allowed for root user with password

After the first login remember to:

- change all default passwords (see below)
- configure NethVoice Proxy before the NethVoice module

## Console/SSH
- **User:** `root`
- **Password:** `Nethesis,1234`

## Cluster
- **User:** `admin`
- **Password:** `Nethesis,1234`

## OpenLDAP
- **User:** `administrator`
- **Password:** `Nethesis,1234`
