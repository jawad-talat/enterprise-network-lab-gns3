# Enterprise Network Lab — GNS3 (Real Cisco IOS)

Same enterprise network topology as the Packet Tracer lab but rebuilt 
on real Cisco IOS using GNS3 and VMware. This version runs actual 
Cisco IOS 12.4 on c3725 routers and IOU L2/L3 images for switching.

---

## Topology

```
PC-Internet -- R1 ======WAN====== R2 -- SW3(Branch)
               |                          |
              SW1(Core)              PC-Branch1
               |                    PC-Branch2
              SW2(Access)
               |
     PC-Sales, PC-IT, PC-HR, PC-Mgmt
```

---

## Devices Used

| Device | Platform | Role |
|--------|----------|------|
| R1 | Cisco c3725 IOS 12.4 | HQ Router |
| R2 | Cisco c3725 IOS 12.4 | Branch Router |
| SW1 | Cisco IOU L2 15.1 | HQ Core Switch |
| SW2 | Cisco IOU L2 15.1 | HQ Access Switch |
| SW3 | Cisco IOU L2 15.1 | Branch Switch |

---

## IP Addressing

| Device | Interface | IP | Subnet |
|--------|-----------|-----|--------|
| R1 | f0/0 | 10.0.0.1 | /30 |
| R1 | f0/1 | 203.0.113.1 | /30 |
| R1 | f1/0.10 | 192.168.10.1 | /26 |
| R1 | f1/0.20 | 192.168.20.1 | /27 |
| R1 | f1/0.30 | 192.168.30.1 | /28 |
| R1 | f1/0.40 | 192.168.40.1 | /29 |
| R2 | f0/0 | 10.0.0.2 | /30 |
| R2 | f0/1.10 | 192.168.110.1 | /27 |
| R2 | f0/1.20 | 192.168.120.1 | /28 |
| SW3 | Vlan20 | 192.168.120.7 | /28 |
| PC-Internet | e0 | 203.0.113.2 | /30 |

---

## Features Implemented

- Multi-VLAN HQ network using Router-on-a-Stick with dot1Q trunking
- OSPF dynamic routing between HQ and Branch
- Centralized DHCP on R1 with DHCP Relay on R2 for Branch VLANs
- NAT/PAT translating all private traffic to 203.0.113.1
- SSH v2 on R1 and R2
- Port Security with sticky MAC on all access ports
- DHCP Snooping with trusted uplinks on all switches
- Extended ACL restricting Sales to ICMP only toward Branch
- Standard ACL blocking HR from reaching Branch entirely

---

## Key Differences from Packet Tracer Version

- Real Cisco IOS 12.4 — actual routing engine not simulation
- IOU L2 switches require manual dot1Q encapsulation before trunking
- No SSH support on IOU L2 image — limitation of that image
- VPCS used for end devices — run `dhcp` command to get IPs
- `no ip domain-lookup` required on all routers to prevent DNS hangs on typos
- 32-bit library support manually installed on GNS3 VM for IOU images

---

## ACL Design

! Sales → Branch: ICMP only, TCP and UDP blocked
```
ip access-list extended ALLOW_ICMP_ONLY
 permit icmp 192.168.10.0 0.0.0.63 host 192.168.110.6
 deny tcp 192.168.10.0 0.0.0.63 host 192.168.110.6
 deny udp 192.168.10.0 0.0.0.63 host 192.168.110.6
 permit ip any any
```

! HR → Branch: blocked entirely
```
ip access-list standard BLOCK_HR
 deny 192.168.30.0 0.0.0.15
 permit any
```

---

## OSPF Configuration

```
router ospf 1
network 10.0.0.0 0.0.0.3 area 0
network 192.168.10.0 0.0.0.63 area 0
network 192.168.20.0 0.0.0.31 area 0
network 192.168.30.0 0.0.0.15 area 0
network 192.168.40.0 0.0.0.7 area 0
```

---

## Author

Jawad Talat — BS Cybersecurity Student | CCNA 
