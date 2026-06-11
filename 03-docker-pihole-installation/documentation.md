& Configuration Documentation

This comprehensive documentation logs every administrative and configuration step executed during this Homelab project—ranging from the initialization of the Proxmox Virtual Machine (VM) to the deployment of a network-wide Pi-hole advertisement blocker.

---

## 1. Base System & Remote Access Configuration

Following the baseline installation of Ubuntu Server, storage verification was performed via the Proxmox console, confirming a successful root directory disk expansion to a total capacity of 46.94 GB. The host was assigned a static local Internet Protocol (IP) address: `192.168.1.11`.

To operate the server in a headless configuration (without a physical or virtual monitor attached), Secure Shell (SSH) remote access was configured. Remote connections are established from the workstation terminal using the following Command Line Interface (CLI) command:

```bash
ssh reuser@192.168.1.11
```
Troubleshooting Note: The initial connection failure was caused by a syntax error, where the system hostname (home) was swapped with the actual administrative username (reuser). Utilizing the correct username successfully resolved the authentication issue.

## 2. Package Optimization & Docker Environment Installation
To ensure system stability and security, the package repository database was updated, followed by a full system upgrade. Subsequently, the Docker runtime engine and its associated Compose plugin utilities were installed:

```Bash
# Update repository index and upgrade all system packages
sudo apt update && sudo apt upgrade -y

# Install the Docker engine and Docker Compose core utilities
sudo apt install docker.io docker-compose -y
```
To optimize the development workflow and eliminate the requirement of prepending sudo (superuser do) administrative privileges to every container management command, the user account was appended to the local Docker security group, and group privileges were instantly refreshed:

```Bash
# Append the 'reuser' account to the Docker system group
sudo usermod -aG docker reuser

# Apply and activate the new group privileges immediately within the current shell session
newgrp docker
```

### 3. Pi-hole DNS Shield Configuration & Troubleshooting
A. Releasing Port 53 (Mitigating Host Resolver Conflicts)
Ubuntu Server deploys a native local stub resolver called systemd-resolved, which binds to port 53 by default. This conflict prevents custom Domain Name System (DNS) containers like Pi-hole from binding to the interface. The native service was permanently halted and masked, and the host's own local resolver was manually redirected to Cloudflare's public DNS architecture:

```Bash
# Halt the active systemd-resolved service daemon
sudo systemctl stop systemd-resolved

# Prevent the systemd-resolved service from initiating during system boot
sudo systemctl disable systemd-resolved

# Remove the existing system-managed symbolic link configuration file
sudo rm /etc/resolv.conf

# Generate a static configuration file pointing to an upstream public DNS resolver
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
```

## Docker Compose Blueprint Construction
Within the ~/homelab/pihole working directory, the docker-compose.yml configuration blueprint was structured. Syntax corrections were applied to remove trailing whitespaces, and the legacy version tag was omitted in compliance with the modern Docker Compose V2 specifications.

The finalized, production-ready YAML Ain't Markup Language (YAML) structural design is defined below:

```YAML
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"    # Standard DNS Query Port over TCP
      - "53:53/udp"    # Standard DNS Query Port over UDP
      - "8080:80/tcp"  # Redirected Web Graphical User Interface (GUI) HTTP Port
    environment:
      TZ: 'Europe/Budapest'                  # Defines the container internal timezone
      WEBPASSWORD: 'reuser_secret_password'  # Sets the administrative web dashboard access password
    volumes:
      - './etc-pihole:/etc/pihole'           # Persistent storage volume for Pi-hole core configurations
      - './etc-dnsmasq.d:/etc/dnsmasq.d'     # Persistent storage volume for custom DNS replication rules
    restart: unless-stopped                  # Ensures container auto-restart policies unless manually killed
```
The container infrastructure was initialized utilizing the modern Docker V2 execution command:

```Bash
docker compose up -d
```
C. Web Authentication & Network Security Permissions (Pi-hole v6)
During the initial initialization phase, the web engine dashboard failed to authenticate using the environment variable password parameter. To circumvent this, direct access was forced into the active container run-time environment using the latest Pi-hole v6 CLI management syntax to overwrite the password string:

```Bash
docker exec -it pihole pihole setpassword
```
Following a successful login, the web interface was toggled to Expert Mode, and the Interface settings policy was modified to Permit all origins.

## 4. Global Network Integration
To achieve seamless, automated network-wide protection without modifying every client workstation individually, the local Dynamic Host Configuration Protocol (DHCP) server settings on the primary residential gateway (router) were modified. The primary DNS Server parameter was directed exclusively to the server IP address (192.168.1.11).

Critical Infrastructure Security Dependency: Disabling IPv6
During router integration, the Internet Protocol Version 6 (IPv6) stack was completely disabled globally across the router.

The Architecture Reason: Modern desktop operating systems and smartphones inherently prioritize IPv6 routing logic over standard Internet Protocol Version 4 (IPv4) connections. If IPv6 remained active, client devices would automatically discover and utilize the upstream Internet Service Provider (ISP) network IPv6 DNS addresses, completely bypassing the IPv4 Pi-hole sinkhole and rendering the advertisement blocking mechanism ineffective.

To force client machines to adapt to the new infrastructure routing policies immediately without waiting for standard expiration timers, a client-side network lease cache flush must be initiated:

```DOS
# Windows command to purge the local DNS resolver cache
ipconfig /flushdns
```
Alternatively, cycling the client device's network interfaces (toggling Wi-Fi off and on, or cycling Airplane Mode for 5 seconds) forces a new DHCP lease negotiation.
