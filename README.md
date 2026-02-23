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

Each VLAN is defined as a `subnet` block in [`dhcpd.conf`](dhcpd.conf). To prevent IP conflicts, ranges are restricted to `.10` through `.240`, leaving the `.1 - .9` and `.241 - .254` ranges for gateways and static infrastructure devices (e.g., the server at `.231`).

The full configuration covers **102 subnets** across seven sectors:

| Sector | Address Range | VLANs |
|--------|--------------|-------|
| Infrastructure | `10.10.x.0/24` | Voice, Security, Server Farms, Computer Centre, GYMHALL |
| Academic Block Labs (A–V) | `10.20.x.0/24` | One wired lab per block |
| Academic Block Wireless (A–V) | `10.21.x.0/24` | One wireless network per block |
| Student Common Areas | `10.30.x.0/24` | Wireless zones, BYOD, Guest, MFDs |
| Staff / Administration | `10.40.x.0/24` | Admin, Finance, HR, ICT, Registrar |
| Remote Campus — Deveyton | `10.100.x.0/24` | Admin, Labs, Staff, Voice |
| Remote Campus — Secunda | `10.60.x.0/24` | Admin, Labs, Staff, Voice |
| Specialized Facilities | `10.50.x.0/24` | Engineering, Science, Design, Health Sciences |

**Example Block (Block A Lab):**

```conf
subnet 10.20.1.0 netmask 255.255.255.0 {
  range 10.20.1.10 10.20.1.240;
  option routers 10.20.1.253;
  option domain-name-servers 192.0.2.1, 192.0.2.2;
}
```

Copy [`dhcpd.conf`](dhcpd.conf) to `/etc/dhcp/dhcpd.conf` on the server to deploy the full configuration.

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
