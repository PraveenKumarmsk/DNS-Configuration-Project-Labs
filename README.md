# Cisco Packet Tracer: DNS Configuration Lab

## 📌 Project Overview
This project demonstrates a comprehensive network configuration in Cisco Packet Tracer. It features a centralized **DHCP Server** that provides IP addresses to two different subnets, and a **DNS Server** that resolves domain names to IP addresses for the clients. 

Because DHCP broadcasts do not cross router boundaries, the router on the right acts as a **DHCP Relay Agent** (`ip helper-address`) to forward requests to the central DHCP server.

![Network Topology](https://github.com/PraveenKumarmsk/DNS-Configuration-Project-Labs/blob/main/DNS%20Lab%20SS.png)

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
```
2. Router1 Configuration (DHCP Relay Agent)
Router1 does not have DHCP pools. Instead, it forwards DHCP broadcast requests from the 192.168.2.0/24 network to the DHCP server at 1.0.0.1.
```
Router> enable
Router# configure terminal
Router(config)# hostname RELAY_AGENT

! Configure LAN Interface
Router(config)# interface FastEthernet0/1
Router(config-if)# ip address 192.168.2.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure WAN Interface
Router(config)# interface FastEthernet0/0
Router(config-if)# ip address 1.0.0.2 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure DHCP Relay (IP Helper Address)
Router(config)# interface FastEthernet0/1
Router(config-if)# ip helper-address 1.0.0.1
Router(config-if)# exit

! Configure Routing (Static Route to Left LAN)
Router(config)# ip route 192.168.1.0 255.255.255.0 1.0.0.1
```
3. DNS Server Configuration (Server GUI)
The DNS server resolves domain names to IP addresses. This is configured via the Packet Tracer GUI.

Click on the DNS SERVER.

Go to the Desktop tab -> IP Configuration.

Set IP: 192.168.1.3

Set Subnet Mask: 255.255.255.0

Set Default Gateway: 192.168.1.1

Set DNS Server: 127.0.0.1 (or leave blank)

Go to the Services tab -> DNS.

Ensure the DNS Service is set to On.

Add DNS Records:

Name: www.test.com | Type: A Record | Address: 192.168.1.2 (Click Add)

Name: tftp.test.com | Type: A Record | Address: 192.168.1.4 (Click Add)

4. Client PC Configuration
Click on CLIENT - 1, CLIENT - 2, or CLIENT - 3.

Go to Desktop -> IP Configuration.

Select DHCP.

Verify they receive an IP address, subnet mask, default gateway, and the DNS Server address (192.168.1.3).

✅ Verification & Testing
Verify DHCP:

On CLIENT - 2 (Right LAN), open the Command Prompt.

Type ipconfig. It should have an IP in the 192.168.2.x range, Gateway 192.168.2.1, and DNS 192.168.1.3.

On the DHCP Server router, run show ip dhcp binding to see the leased addresses.

Verify Routing:

From CLIENT - 2, ping 192.168.1.2 (Web Server). This tests the routing between the two LANs.

From CLIENT - 1, ping 192.168.2.2 (Client 2).

Verify DNS:

On CLIENT - 1 or CLIENT - 2, open the Web Browser (Desktop tab).

Type http://www.test.com in the URL bar and press Enter.

It should successfully connect to the WEB SERVER (assuming the Web Server has a basic HTML page configured in its Services tab).

Alternatively, open Command Prompt and type nslookup www.test.com. It should return 192.168.1.2.

🛠️ Tools Used
Cisco Packet Tracer

DHCP (Dynamic Host Configuration Protocol)

DNS (Domain Name System)

Static Routing

Cisco IOS CLI

📂 How to Use
Clone this repository.

Open the .pkt file in Cisco Packet Tracer.

Review the configurations or use the CLI commands and GUI steps provided above to rebuild the lab from scratch.
