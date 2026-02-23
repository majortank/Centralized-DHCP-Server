# Centralized Campus DHCP Server (Ubuntu Server)

This project configures an **IBM System x3100 M4** tower server to provide dynamic IP addressing across 100+ VLANs for the VDBP campus. The system utilizes **ISC-DHCP-Server** on Ubuntu to handle requests relayed via **IP Helpers** from core switches.

> **Security Note:** All IP addresses shown in this documentation are illustrative examples only and do not reflect the actual network topology.

## 📋 System Specifications

* **Hardware:** IBM System x3100 M4
* **Operating System:** Ubuntu Server (Headless)
* **Network Interface:** `enp11s0` (Physical Port 2)
* **Server Static IP:** `192.0.2.231` (VLAN 15 - GYMHALL)
* **Primary Gateway:** `192.0.2.253`

## 🛠️ Network Architecture

The server is positioned in **VLAN 15** but serves the entire campus infrastructure through a **DHCP Relay** (IP Helper) configuration.

### Key Managed Sectors

The configuration includes specific subnets for:

* **Academic Blocks:** Blocks A through V (e.g., Block A Lab: `198.51.100.0/24`)
* **Infrastructure:** Voice (VLAN 2), Security Devices (VLAN 5), and Server Farms (VLAN 6 & 9)
* **Specialized Labs:** Computer Centre (VLAN 13) and various Student Labs
* **Remote Campuses:** Deveyton (`198.51.100.0/24`) and Secunda (`203.0.113.0/24`)

## 🚀 Configuration Files

### 1. Interface Binding

The service is locked to the physical hardware interface to prevent accidental broadcasts on management ports.

**File:** `/etc/default/isc-dhcp-server`

```bash
INTERFACESv4="enp11s0"
```

### 2. DHCP Declarations

Each VLAN is defined as a `subnet` block. To prevent IP conflicts, ranges are generally restricted to `.10` through `.250`, leaving the `.1 - .9` and `.251 - .254` ranges for gateways and infrastructure.

**Example Block (Block A):**

```conf
subnet 198.51.100.0 netmask 255.255.255.0 {
  range 198.51.100.10 198.51.100.250;
  option routers 198.51.100.253;
  option domain-name-servers 192.0.2.1, 192.0.2.2;
}
```

## 🔧 Maintenance Commands

**Test configuration for syntax errors:**

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

**Restart the DHCP service:**

```bash
sudo systemctl restart isc-dhcp-server
```

**Monitor real-time IP assignments:**

```bash
tail -f /var/log/syslog | grep dhcpd
```
