Markdown
# Automated SME Campus Infrastructure Deployment with Cisco ISR 4331

## 🛠️ Project Overview
This project demonstrates the deployment of a scalable, automated corporate network architecture tailored for Small-to-Medium Enterprises (SMEs). Developed within **Cisco Packet Tracer**, this infrastructure implements dynamic host configuration (DHCP) across distinct operational segments, optimizing local network topologies to handle both fixed workplace nodes and wireless mobile endpoints efficiently.

The topology successfully eliminates manual addressing overhead and resolves two critical production-level network failures during the deployment phase.

---

## 📐 Network Architecture & Design Strategy

The topology segments the corporate enterprise environment into clear functional zones, managed directly via a centralized multi-interface Cisco ISR 4331 Router.

### 1. Operational Segments
* **Wired Core Infrastructure (LAN Segment):** Structured with a standard Class C address spacing mapped onto `192.168.1.0/24`. This partition terminates static enterprise units including administrative desktop terminals (`PC0`-`PC3`) and the local network printer (`Printer0`).
* **Wireless Fleet Mobility Zone (WLAN Segment):** Integrated into the same logical broadcast domain to maintain unified network access. It aggregates dynamic corporate endpoints including laptops (`Laptop0`-`Laptop3`), mobile tablets, and an on-site **Wireless Printer** executing wireless over-the-air jobs via a centralized **Access Point**.

### 2. Interface Layer Mapping
* **`GigabitEthernet0/0/1`:** Aggregates local frame traffic traversing `Switch0`. Serves as the **Default Gateway** (`192.168.1.1`) for the entire enterprise workstation environment.
* **`GigabitEthernet0/0/0`:** Dedicated link for critical local storage clusters (`Server0`), configured with a static IP reservation.

---

## ⚙️ Cisco IOS Configuration Blueprint

### 1. Hardening Interface Gates & Base IP Definitions
```text
Router> enable
Router# configure terminal
Router(config)# hostname Core-ISR4331

! -- Configuring Wired Production Interface --
Core-ISR4331(config)# interface GigabitEthernet0/0/1
Core-ISR4331(config-if)# description LAN_WIRED_WORKSTATIONS
Core-ISR4331(config-if)# ip address 192.168.1.1 255.255.255.0
Core-ISR4331(config-if)# no shutdown
Core-ISR4331(config-if)# exit

! -- Configuring Local Server Interface --
Core-ISR4331(config)# interface GigabitEthernet0/0/0
Core-ISR4331(config-if)# description LOCAL_STORAGE_SERVER
Core-ISR4331(config-if)# ip address 172.16.0.1 255.255.0.0
Core-ISR4331(config-if)# no shutdown
Core-ISR4331(config-if)# exit

2. Address Exclusion Protocols (Preventing IP Collisions)
Critical structural hardware assets utilize hard-coded static addresses. The router is explicitly ordered to withhold these specific slots from standard dynamic leases.
! -- Reserving LAN Default Gateway Scope –
 Core-ISR4331(config)# ip dhcp excluded-address 192.168.1.1 
! -- Reserving Server Network Gateway and Critical Storage IP –
 Core-ISR4331(config)# ip dhcp excluded-address 172.16.0.1 
Core-ISR4331(config)# ip dhcp excluded-address 172.16.0.100

3. State-Aware DHCP Address Space Deployment
 -- Constructing Address Pool for Station Workstations & Wireless Fleet --
Core-ISR4331(config)# ip dhcp pool LAN_PRODUCTION_POOL
Core-ISR4331(dhcp-config)# network 192.168.1.0 255.255.255.0
Core-ISR4331(dhcp-config)# default-router 192.168.1.1
Core-ISR4331(dhcp-config)# end

! -- Writing Running Configuration to NVRAM --
Core-ISR4331# write memory
 Field Troubleshooting Journal (Incident Reports)
Incident 1: Fatal ROMMON (ROM Monitor) Boot Defect
•	Observation: Upon manual modification of physical module interfaces, the Cisco terminal crashed directly into non-volatile boot loader sub-routines, rendering common global execution commands completely unresponsive with a not found error output.
•	Root Cause: Modification of hardware components while hot-powered triggered an abrupt initialization interruption, forcing the hardware system to skip loading the standard Cisco IOS binary from flash media.
•	Resolution: Forced diagnostic hardware reload by feeding the terminal low-level boot instructions:
rommon 1> boot
Followed by a controlled power cycle to correctly remount non-volatile elements safely into normal operational states.


Incident 2: Host APIPA Failure Over Wireless Terminals (169.254.X.X)
•	Observation: Wireless endpoints successfully bound to raw hardware layer channels but immediately generated broken local allocation spaces starting within 169.254.X.X network scopes.
•	Root Cause: Initial deployment of high-level Enterprise Wireless LAN Controllers (WLC) introduced advanced encapsulation configurations incompatible with simple localized flat-network scopes. The missing downstream pathway to the DHCP daemon triggered operating system fallback protocols (APIPA).
•	Resolution: Removed heavy enterprise controller hardware. Substituted with an unencapsulated Access Point-PT configured with standard WMP300N Wi-Fi network cards. Tied infrastructure gates into centralized aggregation switches via standard category straight-through copper elements.
 Verification Mechanics & Validation Proof
To verify network validity, execute validation checks on the Cisco CLI to verify concurrent host leases across local networks:
Core-ISR4331# show ip dhcp binding
Expected Outputs mapping MAC targets cleanly to authorized network zones prove global convergence and reliable end-to-end routing.

