# Homelab Server Deployment & Pi-hole DNS Configuration Guide

This documentation covers the step-by-step process of resolving network port conflicts on Linux, configuring client-side Domain Name System (DNS) settings, testing network propagation, and analyzing deployment troubleshooting history.

---

## 1. Linux Host Network Optimization (Resolving Port 53 Conflicts)

By default, Ubuntu Server utilizes a local stub resolver that occupies port 53. To deploy a custom DNS container like Pi-hole, this service must be stopped and disabled.

Execute the following commands in your Command Line Interface (CLI):

```bash
# 1. Stop the active systemd-resolved service immediately
sudo systemctl stop systemd-resolved

# 2. Prevent the service from starting automatically during system boot
sudo systemctl disable systemd-resolved

# 3. Remove the existing symbolic link or configuration file
sudo rm /etc/resolv.conf

# 4. Create a new static configuration file pointing to an upstream DNS (Cloudflare)
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
```

## 2. Windows Client Configuration (Directing Traffic to Pi-hole)
To route your client workstation's network traffic through the newly established Pi-hole shield, manual Internet Protocol Version 4 (TCP/IPv4) settings must be applied.

Press Windows + R on your keyboard to initiate the Run dialog box.

Input ncpa.cpl and press Enter to instantly access the legacy Network Connections Graphical User Interface (GUI).

Identify your active network interface card (Ethernet or Wi-Fi), right-click, and select Properties.

Double-click on Internet Protocol Version 4 (TCP/IPv4) from the itemized list.

Toggle the selection to: Use the following DNS server addresses.

Input your home server's static Internet Protocol (IP) address into the Preferred DNS server field:
192.168.1.11

Leave the Alternate DNS server field entirely blank. This forces the operating system to route all queries exclusively through the Pi-hole instance.

Click OK on both windows to commit and apply changes.

## 3. Verification & Live Network Testing
Once the client-side network pipeline is adjusted, execute a live test to verify the DNS sinkhole efficiency:

Open a web browser on the configured client machine and visit several high-traffic news platforms or run search queries.

Access the Pi-hole web management dashboard interface and refresh the page (F5).

Verification Metrics: * The Total Queries metric counter will sharply increase.

The Queries Blocked metric will begin counting restricted advertisements and telemetry trackers in real-time.

## 4. Crucial Networking Architecture & Insights
A. DNS Propagation and DHCP Lease Management
When altering DNS settings at the router level, client devices (smartphones, Smart TVs, laptops) do not reflect the changes instantaneously. This is because clients operate under pre-existing Dynamic Host Configuration Protocol (DHCP) leases.

The Resolution: It is unnecessary to reboot the entire network router. Instead, toggle the client device's Wi-Fi interface off and on, or cycle through Airplane Mode for 5 seconds. This forces the device to request a new DHCP lease, obtaining the updated Pi-hole DNS server address.

[Hungarian Context / Magyar magyarázat]

Network propagation: Hálózati elterjedés / frissülés.

DHCP lease: Hálózati bérleti szerződés. A router bizonyos időre adja oda az IP és DNS adatokat a gépnek, ami lejáratig nem kér újat. A Wi-Fi ki-be kapcsolásával ezt a bérletet kényszerítjük frissítésre.

## 5. The YouTube Ad-Blocking Framework Constraint
A common misconception in homelab infrastructure is that a DNS-level sinkhole can neutralize native YouTube application advertisements.

The Root Cause: Standard websites host advertising content on dedicated third-party domains (e.g., ads.example.com), allowing Pi-hole to easily filter them. Conversely, Google delivers YouTube advertisements from the exact same primary domains and content delivery servers as the video stream itself. Restricting the advertisement domain would inadvertently break the core video playback stream.

Alternative Solutions: Client-side application-layer filtering is mandatory for YouTube ad mitigation. Recommended alternatives include the Brave Browser, the uBlock Origin browser extension, or specialized application packages like YouTube ReVanced.
