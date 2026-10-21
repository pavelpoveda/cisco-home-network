# Network Verification

This document records the checks I completed on the physical Cisco lab. Tests are marked as **Passed**, **Configured**, or **Pending** so the results are clear.

## Tests Completed

| Test | Command or method | Result |
|---|---|---|
| VLAN gateways | R1 `show ip interface brief` | **Passed** — VLANs 10, 20, 30, and 40 were up/up |
| Trunk | SW1 `show interfaces trunk` | **Passed** — Gi0/13 carried VLANs 10, 20, 30, 40 |
| VLAN assignments | SW1 `show vlan brief` | **Passed** — ports assigned to the planned VLANs |
| SSH management | SSH from management workstation | **Passed** — connected to R1 and SW1 |
| Server VLAN connectivity | Ping from Linux test device to `192.168.30.1` | **Passed** — 4 replies, 0% loss |
| Internet connectivity | Ping to `1.1.1.1` from Linux test device | **Passed** — 4 replies, 0% loss, route through R1 |
| Client ACL permitted traffic | SSH from admin workstation to VLAN 30 test device | **Passed** — SSH and internet still worked after applying `CLIENTS-IN` |
| DHCP Snooping, VLAN 20 | SW1 `show ip dhcp snooping binding` | **Passed** — binding learned on Gi0/1 for a VLAN 20 workstation |
| DAI, VLAN 20 | SW1 `show ip arp inspection statistics vlan 20` | **Passed** — 2 forwarded, 0 dropped; 1 DHCP permit |
| NTP | NTP status on R1 and SW1 | **Passed** — switch synchronized with router; router with upstream time source |
| Guest connectivity | Guest test device | **Passed before DAI change** — guest connection and isolation tested; recheck required |

## Security Configuration

**R1:**

- `SSH-MANAGEMENT` limits remote management to approved source addresses.
- `GUEST-IN` blocks guest IPv4 access to private IPv4 networks while allowing other IPv4 traffic.
- `CLIENTS-IN` allows administrative SSH from the designated workstation to the test server, blocks other VLAN 20-to-30 IPv4 traffic, and permits other destinations.

**SW1:**

- DHCP Snooping is enabled on VLANs 20 and 40.
- DAI is enabled on VLANs 20 and 40.
- The router uplink, Gi0/13, is trusted for DHCP Snooping and DAI.
- DHCP Option 82 insertion is disabled.
- Unused access ports are shut down; end-device ports use PortFast and BPDU Guard.

These ACLs are IPv4 filters, not a complete stateful firewall. DAI on VLAN 30 was not enabled because that VLAN uses a static-addressed test device.

## Tests Still Pending

| Test | Why it is pending |
|---|---|
| Blocked VLAN 20-to-30 connection | Requires a separate, non-administrative client on VLAN 20 |
| Guest DHCP and DAI after the latest change | VLAN 40 DAI is configured, but a new guest-device test has not been recorded |
| DHCP Snooping after SW1 restart | Learned bindings may need to be rebuilt or restored after reboot |

## Commands Used

```text
R1# show ip interface brief
R1# show ip access-lists
R1# show running-config interface GigabitEthernet0/0/1.20
SW1# show interfaces trunk
SW1# show vlan brief
SW1# show ip dhcp snooping
SW1# show ip dhcp snooping binding
SW1# show ip arp inspection
SW1# show ip arp inspection statistics vlan 20
```

The configuration files in this repository are sanitized copies. Sensitive device and client identifiers are not included in the public test results.
