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
Replace YOUR_GITHUB_USERNAME and REPO_NAME with your GitHub details.

Make the Script Executable:
bash

Collapse

Wrap

Run

Copy
chmod +x makevpn.sh
Run the Script:
Execute as root:
bash

Collapse

Wrap

Run

Copy
sudo bash makevpn.sh
Follow the interactive prompts:
IP Address: Enter the IPv4 address of your server’s network interface (default: detected eth0 IP or external IP via curl ifconfig.me).
OpenVPN Port: Specify the port for OpenVPN (default: 1194; change if conflicted, e.g., to 1195).
DNS Provider: Choose a DNS provider (default: Google; options include system resolvers, OpenDNS, NTT, Hurricane Electric, Verisign).
Client Name: Provide a name for the initial client certificate (default: client; use one word, no special characters).
If the server is behind NAT, enter the external IP when prompted (leave blank if not NATed).
Verify Installation:
Check OpenVPN service status:
bash

Collapse

Wrap

Run

Copy
systemctl status openvpn@server.service
If it fails, view logs:
bash

Collapse

Wrap

Run

Copy
journalctl -xeu openvpn@server.service
Verify firewall rules:
bash

Collapse

Wrap

Run

Copy
iptables -L -v -n
iptables -t nat -L -v -n
cat /etc/iptables/rules.v4
Ensure DNAT rules exist for VPN client ports (10.8.0.2).
Check OpenVPN files:
bash

Collapse

Wrap

Run

Copy
ls -l /etc/openvpn/{ca.crt,issued/server.crt,private/server.key,dh.pem,ta.key,crl.pem}
Confirm presence of server.conf, ca.crt, dh.pem, ta.key, crl.pem, and issued/ and private/ directories.
Locate client configuration:
bash

Collapse

Wrap

Run

Copy
ls -l ~/client.ovpn ~/client.conf
Test Connectivity:
VPS Ports:
Test external access to TCP 22, 1194, 2222, 10000; UDP 53, 1194, 10000 using nc, telnet, or nmap from another host.
Example:
bash

Collapse

Wrap

Run

Copy
nc -v -u YOUR_SERVER_IP 1194
VPN Client Ports:
Copy ~/client.ovpn to a client device (e.g., PC, phone) and connect using an OpenVPN client (e.g., OpenVPN Connect).
Test access to 10.8.0.2 on TCP 80, 222, 5200, 8000-9600, 15425-15428, 5038, 12010, 22220; UDP 2074-2094, 4569, 4570-4599, 5198-5199, 44966, 12010, 12345, 12346, 20001-20006, 30001, 30050-30056, 40000, 42000, 51100, 52100, 53100, 54100, 62030-62033.
For HamVoIP: Ensure services (e.g., Asterisk on 5060/UDP, RTP on 10000-20000/UDP) are running on 10.8.0.2 and accessible via the VPN.
Example:
bash

Collapse

Wrap

Run

Copy
nc -v 10.8.0.2 5038
Persistence:
Reboot the server:
bash

Collapse

Wrap

Run

Copy
reboot
Recheck OpenVPN and firewall:
bash

Collapse

Wrap

Run

Copy
systemctl status openvpn@server.service
iptables -L -v -n
Manage Clients:
To add a new client:
Re-run the script: sudo bash makevpn.sh.
Select option 1) Add a cert for a new user.
Enter a new client name (e.g., client2).
Retrieve the new .ovpn file from ~/client2.ovpn.
To revoke a client:
Select option 2) Revoke existing user cert.
Choose the client to revoke.
To uninstall OpenVPN:
Select option 3) Remove OpenVPN.
Important Notes
Port Conflicts:
If port 1194 (UDP) is in use, check with:
bash

Collapse

Wrap

Run

Copy
ss -uln | grep 1194
Change the port during script setup or edit /etc/openvpn/server.conf and /root/firewall (replace 1194 with, e.g., 1195), then:
bash

Collapse

Wrap

Run

Copy
/root/firewall restart
systemctl restart openvpn@server.service
Firewall and NAT:
The script uses iptables-nft via iptables-persistent. Ensure no other firewall (e.g., ufw) conflicts:
bash

Collapse

Wrap

Run

Copy
ufw status
ufw disable
For NATed servers, verify the external IP in ~/client.ovpn matches your public IP.
HamVoIP Integration:
Ensure the VPN client (10.8.0.2) runs services on the forwarded ports (e.g., Asterisk on 5060/UDP may need additional forwarding if required).
Test SIP phone connectivity (e.g., Grandstream GRP2612W) via the VPN to the HamVoIP node.
Security:
Change default passwords in /etc/openvpn/server.conf or client configs if modified.
Restrict SSH access (port 22) to trusted IPs if possible:
bash

Collapse

Wrap

Run

Copy
nano /etc/ssh/sshd_config
Backup configurations:
bash

Collapse

Wrap

Run

Copy
tar -czf /root/openvpn-backup-$(date +%F).tar.gz /etc/openvpn /root/firewall /etc/systemd/system/firewall.service /root/*.ovpn /root/client.conf
Troubleshooting:
If OpenVPN fails to start, check logs:
bash

Collapse

Wrap

Run

Copy
journalctl -xeu openvpn@server.service
If ports don’t forward, verify iptables:
bash

Collapse

Wrap

Run

Copy
iptables -t nat -L PREROUTING -v -n
iptables -L FORWARD -v -n
Ensure the VPN client (10.8.0.2) has services listening on forwarded ports.
For file issues, confirm permissions:
bash

Collapse

Wrap

Run

Copy
ls -l /etc/openvpn/
chown nobody:nogroup /etc/openvpn/crl.pem
chmod 644 /etc/openvpn/{ca.crt,issued/*.crt,dh.pem,ta.key}
chmod 600 /etc/openvpn/private/*.key
Community Contribution:
This script was developed with the ham radio community in mind. Share feedback or improvements via GitHub issues or pull requests to benefit hams worldwide.
Consider documenting your HamVoIP setup (e.g., SIP phone integration) alongside this script for a comprehensive guide.
License
MIT License (see LICENSE file for details).

Credits
Author: [Your Name or GitHub Handle]
Contributor: Grok (xAI), for scripting assistance and debugging.
Inspiration: The ham radio community, particularly HamVoIP and AllStarLink users.
text


