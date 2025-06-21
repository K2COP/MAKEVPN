# MAKEVPN
Install script to create a VPN tailored for Ham Radio Nodes running  Allstar Echolink IRLP on ASL3 or HamVoip on Debian 12 Bookworm or Ubuntu 24.04 LTS 


# OpenVPN Road Warrior Installer for HamVoIP and General Use

## Introduction
The `makevpn.sh` script is a robust, user-friendly tool designed to install and configure an OpenVPN server on Ubuntu 24.04 LTS and Debian 12 systems. Tailored for ham radio operators using HamVoIP nodes, it also serves general VPN needs, providing secure, remote access to networks with custom port forwarding. The script automates OpenVPN setup, certificate generation, and firewall configuration, ensuring a seamless experience for connecting clients (e.g., SIP phones, nano nodes, or other devices) to an AllStarLink node or internal services. It includes specific port forwarding rules to support HamVoIP applications, making it ideal for hams seeking to integrate VoIP and VPN technologies, as well as anyone needing a reliable VPN server.

Developed with input from the ham radio community and refined through rigorous testing, this script addresses common VPN challenges like port forwarding and persistence, offering a plug-and-play solution for both novice and experienced users. Whether you’re securing a HamVoIP node or creating a general-purpose VPN, this script simplifies the process while maintaining flexibility and security.

## Features
- **Automated OpenVPN Installation**: Installs OpenVPN and dependencies, including EasyRSA 3.2.0 for secure certificate management.
- **Custom Firewall Configuration**: Configures `iptables` with specific ports for the VPS and VPN client, saved persistently via `iptables-persistent`.
- **Port Forwarding for HamVoIP**:
  - **VPS Ports** (external interface, `eth0`): TCP 22, 1194, 2222, 10000; UDP 53, 1194, 10000.
  - **VPN Client Ports** (10.8.0.2): TCP 80, 222, 5200, 8000-9600, 15425-15428, 5038, 12010, 22220 (maps to 22); UDP 2074-2094, 4569, 4570-4599, 5198-5199, 44966, 12010, 12345, 12346, 20001-20006, 30001, 30050-30056, 40000, 42000, 51100, 52100, 53100, 54100, 62030-62033.
- **Client Management**: Supports adding new client certificates, revoking existing ones, and generating `.ovpn` files for easy client setup.
- **Nano Node Support**: Copies client configuration to `client.conf` for use with nano nodes or other HamVoIP applications.
- **Flexible Configuration**: Prompts for custom IP address, OpenVPN port, DNS provider, and client name during setup.
- **NAT Detection**: Automatically detects NAT environments and allows manual entry of external IP for client connections.
- **Security and Persistence**: Runs OpenVPN as `nobody:nogroup`, uses AES-256-GCM cipher, and ensures firewall rules persist across reboots via a custom systemd service.
- **Tested Compatibility**: Verified to work flawlessly on Ubuntu 24.04 LTS and Debian 12, with modern OpenVPN (2.5+) and `iptables-nft`.

## Supported Distributions
- **Ubuntu 24.04 LTS**
- **Debian 12**

## Instructions for Use

### Prerequisites
- A fresh installation of Ubuntu 24.04 LTS or Debian 12 (minimal or server edition recommended).
- Root access to the system (run the script as `root` or via `sudo`).
- Network connectivity with a primary interface (default: `eth0`) and a public or NATed IP address.
- Basic familiarity with SSH and terminal commands (e.g., `nano`, `chmod`).
- For HamVoIP users: A working AllStarLink node on a Raspberry Pi or similar device, with services (e.g., Asterisk) listening on the VPN client IP (10.8.0.2).

### Installation Steps
1. **Download the Script**:
   - Download `makevpn.sh` from this repository or copy it to your server:
     ```bash
     wget https://raw.githubusercontent.com/YOUR_GITHUB_USERNAME/REPO_NAME/main/makevpn.sh
