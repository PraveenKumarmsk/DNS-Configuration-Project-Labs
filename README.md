# DNS-Lab
# Cisco Packet Tracer: DHCP and DNS Configuration Lab

## 📌 Project Overview
This project demonstrates a comprehensive network configuration in Cisco Packet Tracer. It features a centralized **DHCP Server** that provides IP addresses to two different subnets, and a **DNS Server** that resolves domain names to IP addresses for the clients. 

Because DHCP broadcasts do not cross router boundaries, the router on the right acts as a **DHCP Relay Agent** (`ip helper-address`) to forward requests to the central DHCP server.

## 🗺️ Network Topology & Addressing Scheme

### Network Segments
*   **Left LAN (Headquarters):** `192.168.1.0/24`
*   **WAN Link (Router to Router):** `1.0.0.0/30`
*   **Right LAN (Branch):** `192.168.2.0/24`

### Device IP Assignments

| Device | Interface | IP Address | Subnet Mask | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Router0** (DHCP Server) | Fa0/0 | 1.0.0.1 | 255.255.255.252 | WAN Link to Relay Agent |
| | Fa0/1 | 192.168.1.1 | 255.255.255.0 | Gateway for Left LAN |
| **Router1** (Relay Agent) | Fa0/0 | 1.0.0.2 | 255.255.255.252 | WAN Link to DHCP Server |
| | Fa0/1 | 192.168.2.1 | 255.255.255.0 | Gateway for Right LAN |
| **WEB SERVER** | Fa0 | 192.168.1.2 | 255.255.255.0 | Static IP |
| **DNS SERVER** | Fa0 | 192.168.1.3 | 255.255.255.0 | Static IP |
| **TFTP SERVER** | Fa0 | 192.168.1.4 | 255.255.255.0 | Static IP |
| **CLIENT - 1** | Fa0 | DHCP | DHCP | Assigned by Router0 |
| **CLIENT - 2** | Fa0 | DHCP | DHCP | Assigned by Router0 |
| **CLIENT - 3** | Fa0 | DHCP | DHCP | Assigned by Router0 |

---

## ⚙️ Configuration Steps

### 1. Router0 Configuration (DHCP Server & Gateway)
Router0 is configured to hand out IP addresses for both the 192.168.1.0/24 and 192.168.2.0/24 networks. It also acts as the DNS entry point for the clients.

```bash
Router> enable
Router# configure terminal
Router(config)# hostname DHCP_SERVER

! Configure LAN Interface
Router(config)# interface FastEthernet0/1
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure WAN Interface
Router(config)# interface FastEthernet0/0
Router(config-if)# ip address 1.0.0.1 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure DHCP Pool for Left LAN (192.168.1.0/24)
Router(config)# ip dhcp pool LEFT_LAN
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 192.168.1.3
Router(dhcp-config)# exit

! Configure DHCP Pool for Right LAN (192.168.2.0/24)
Router(config)# ip dhcp pool RIGHT_LAN
Router(dhcp-config)# network 192.168.2.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.2.1
Router(dhcp-config)# dns-server 192.168.1.3
Router(dhcp-config)# exit

! Exclude Static IP addresses from DHCP assignment
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10

! Configure Routing (Static Route to Right LAN)
Router(config)# ip route 192.168.2.0 255.255.255.0 1.0.0.2
