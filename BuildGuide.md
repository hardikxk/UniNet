# Build Guide: Secure Multi-Campus University Network

> **Revision notes (corrected version).** This copy of the guide includes everything that was found while building the project in Packet Tracer:
>
> - **Step 2:** the serial module ports on your routers are `Serial0/3/0` and `Serial0/3/1`, not `Serial0/0/0` and `Serial0/0/1`.
> - **Step 3:** every cable now has an explicit cable type and an explicit port on both ends, and there is a CDP check to prove the wiring. Do **not** use Automatic connection mode; it picks the first free port and shifts cables.
> - **Step 4:** a new interface-to-address map ties each port to its IP address.
> - **Step 5:** the 3560 needs `switchport trunk encapsulation dot1q` before `switchport mode trunk`. The original block left this out, so the trunks never formed.
> - **Step 6:** commands unchanged. A tip was added for clearing stale port-security entries if cables were moved after configuring.
> - **Step 7 and Step 13:** serial interface names changed to `0/3/0` and `0/3/1`.
> - **Step 15:** extra troubleshooting rows for the problems above.

## 1. Before you begin

Save the Packet Tracer file immediately as `University_MultiCampus_Final_Project.pkt`. Save again after each completed stage. Use **Realtime** mode while configuring; use **Simulation** mode only for demonstrations.

This guide uses Cisco Packet Tracer device names and interfaces commonly available on the 2811 router, 3560 multilayer switch, and 2960 switch. If Packet Tracer displays a different interface label, use the matching physical port and substitute that label consistently in the commands.

### Project conventions

- CLI commands shown after `#` are entered from privileged EXEC mode.
- Commands shown after `(config)#` or `(config-if)#` are entered in the relevant configuration mode.
- Type `enable` to change `>` into `#`.
- Type `configure terminal` to enter global configuration mode.
- Type `end` to return to `#`.
- Save each device with `copy running-config startup-config`; press **Enter** when it asks for the destination filename.
- Choose the cable type and **both ports yourself** for every link (see Step 3). Do not use **Automatically Choose Connection Type**, because it picks the first free port and shifts cables.
- Use **Serial DCE** only for the four WAN links listed in Step 3.

## 2. Add the devices

Open a blank Packet Tracer file and place the following devices. Rename a device by clicking it, opening the **Config** tab, and editing **Display Name**.

### Routers

From **Network Devices > Routers**, place four **2811** routers and rename them:

1. `R-HQ`
2. `R-Engineering`
3. `R-Administration`
4. `R-Hostel`

#### Install serial modules

Do this on every 2811 router before making serial connections:

1. Click the router.
2. Open the **Physical** tab.
3. Turn the power switch off.
4. Drag a **WIC-2T** module into an empty slot.
5. Turn the power switch on.
6. Wait until the router restarts.

The two serial ports appear as `Serial0/3/0` and `Serial0/3/1` when the module lands in HWIC slot 3 (this is what happens in this project). Confirm with `show ip interface brief`. If your router shows different serial names, substitute them consistently everywhere this guide says `Serial0/3/0` or `Serial0/3/1`. If you see `% Invalid interface type and number` when entering `interface serial...`, the label is wrong.

### Switches

From **Network Devices > Switches**, place:

- One **3560-24PS Multilayer Switch** named `CORE-HQ`.
- Three **2960** switches named `SW-Admin`, `SW-Faculty`, and `SW-Students`.
- Three **2960** switches named `SW-Engineering`, `SW-Administration`, and `SW-Hostel`.

### End devices and servers

From **End Devices**, place:

#### PCs
- `Admin-PC1`, `Admin-PC2`, `Admin-PC3`
- `Faculty-PC1`, `Faculty-PC2`, `Faculty-PC3`
- `Student-PC1`, `Student-PC2`, `Student-PC3`, `Student-PC4`
- `Eng-PC1`, `Eng-PC2`, `Eng-PC3`
- `BranchAdmin-PC1`, `BranchAdmin-PC2`, `BranchAdmin-PC3`
- `Hostel-PC1`, `Hostel-PC2`, `Hostel-PC3`

#### Servers
- Three **Server** devices named `Server-DNS-WEB`, `Server-FTP`, and `Server-MAIL`

Arrange the project visually: headquarters at the top centre, branches around the lower portion, and WAN serial links in a ring. Place each branch's PCs next to its switch so the cables stay short and readable. Add labels using the **Place Note** tool if useful.

## 3. Make every connection

There are **33 cables** in total: 7 on the HQ core, 10 from HQ PCs to access switches, 3 branch router-to-switch links, 9 branch PCs, and 4 WAN serial links.

### 3.1 Which cable type to use

Use straight-through between different kinds of device and cross-over between the same kind (switch to switch).

| Link | Cable type (Connections tool) | Line style |
|---|---|---|
| `CORE-HQ` to HQ access switches | **Copper Cross-Over** | dashed |
| `CORE-HQ` to `R-HQ` | **Copper Straight-Through** | solid |
| `CORE-HQ` to servers | **Copper Straight-Through** | solid |
| Access switch to PC | **Copper Straight-Through** | solid |
| Branch router to branch switch | **Copper Straight-Through** | solid |
| Router to router (WAN) | **Serial DCE** | red zig-zag |

### 3.2 How to place one cable

1. Click the lightning-bolt **Connections** icon and choose the cable type from the table above.
2. Click the first device. A list of its ports appears; choose the exact port from the table below.
3. Click the second device and choose its exact port.
4. To remove a wrong cable, use the **Delete** tool (red X) and click the cable, then redraw it.
5. Hover over a cable end at any time to see which port it is on.

### 3.3 HQ core switch cables (7)

| # | First device and port | Second device and port | Cable | Purpose |
|---:|---|---|---|---|
| 1 | `CORE-HQ Fa0/1` | `R-HQ Fa0/0` | Straight-Through | Routed HQ core-to-router link |
| 2 | `CORE-HQ Fa0/2` | `SW-Admin Fa0/24` | Cross-Over | Admin trunk |
| 3 | `CORE-HQ Fa0/3` | `SW-Faculty Fa0/24` | Cross-Over | Faculty trunk |
| 4 | `CORE-HQ Fa0/4` | `SW-Students Fa0/24` | Cross-Over | Student trunk |
| 5 | `CORE-HQ Fa0/5` | `Server-DNS-WEB Fa0` | Straight-Through | Server VLAN |
| 6 | `CORE-HQ Fa0/6` | `Server-FTP Fa0` | Straight-Through | Server VLAN |
| 7 | `CORE-HQ Fa0/7` | `Server-MAIL Fa0` | Straight-Through | Server VLAN |

Important: the uplink on each access switch is **Fa0/24**, not Fa0/1. Fa0/1 on the access switches is a PC port.

### 3.4 HQ access switch PC cables (10, all Straight-Through)

The PC end is always the PC's `FastEthernet0` (`Fa0`).

| Switch port | PC | VLAN |
|---|---|---|
| `SW-Admin Fa0/1` | `Admin-PC1` | 10 |
| `SW-Admin Fa0/2` | `Admin-PC2` | 10 |
| `SW-Admin Fa0/3` | `Admin-PC3` | 10 |
| `SW-Faculty Fa0/1` | `Faculty-PC1` | 20 |
| `SW-Faculty Fa0/2` | `Faculty-PC2` | 20 |
| `SW-Faculty Fa0/3` | `Faculty-PC3` | 20 |
| `SW-Students Fa0/1` | `Student-PC1` | 30 |
| `SW-Students Fa0/2` | `Student-PC2` | 30 |
| `SW-Students Fa0/3` | `Student-PC3` | 30 |
| `SW-Students Fa0/4` | `Student-PC4` | 30 |

The Step 6 configuration shuts down every unused port above these ranges. A PC plugged into `Fa0/4` or higher on SW-Admin or SW-Faculty (`Fa0/5` or higher on SW-Students) will show a **red** link marker and will not work.

### 3.5 Branch cables (12, all Straight-Through)

Each branch router connects to its switch on **Fa0/24**, and the three PCs use **Fa0/1** to **Fa0/3**.

| Branch | Router port | Switch uplink port | PC ports on the switch |
|---|---|---|---|
| Engineering | `R-Engineering Fa0/0` | `SW-Engineering Fa0/24` | `Fa0/1` Eng-PC1, `Fa0/2` Eng-PC2, `Fa0/3` Eng-PC3 |
| Administration | `R-Administration Fa0/0` | `SW-Administration Fa0/24` | `Fa0/1` BranchAdmin-PC1, `Fa0/2` BranchAdmin-PC2, `Fa0/3` BranchAdmin-PC3 |
| Hostel | `R-Hostel Fa0/0` | `SW-Hostel Fa0/24` | `Fa0/1` Hostel-PC1, `Fa0/2` Hostel-PC2, `Fa0/3` Hostel-PC3 |

The PC end is `Fa0` on each PC.

### 3.6 WAN serial ring (4)

Choose **Serial DCE** for each row. Click the DCE router first; the first listed side is the clocking (DCE) side.

| # | DCE first side | DTE second side | Router that needs `clock rate 64000` |
|---:|---|---|---|
| 1 | `R-HQ Serial0/3/0` | `R-Engineering Serial0/3/0` | R-HQ |
| 2 | `R-Engineering Serial0/3/1` | `R-Administration Serial0/3/0` | R-Engineering |
| 3 | `R-Administration Serial0/3/1` | `R-Hostel Serial0/3/0` | R-Administration |
| 4 | `R-Hostel Serial0/3/1` | `R-HQ Serial0/3/1` | R-Hostel |

If a serial interface name differs, check it with `show ip interface brief` and use the actual label. You can confirm DCE/DTE with `show controllers serial 0/3/0`.

### 3.7 Verify the cabling before you configure anything

CDP works without any IP configuration, so it is a good wiring check. On `CORE-HQ`, run `show cdp neighbors`. Once the access switches and `R-HQ` are powered, it should list exactly these (servers do not appear in CDP):

| CORE-HQ local port | Neighbour | Neighbour port |
|---|---|---|
| `Fas 0/1` | `R-HQ` | `Fas 0/0` |
| `Fas 0/2` | `SW-Admin` | `Fas 0/24` |
| `Fas 0/3` | `SW-Faculty` | `Fas 0/24` |
| `Fas 0/4` | `SW-Students` | `Fas 0/24` |

On each branch switch, `show cdp neighbors` should show the branch router on `Fa0/24`.

Stale entries from old cables can stay listed for a few minutes (hold time counting down). Once the cables are correct they disappear on their own.

To check the PC and server ports after the switches are configured (Steps 5, 6 and 8), run `show interfaces status` on each switch. The PCs must be `connected` on `Fa0/1` to `Fa0/3` (`Fa0/1` to `Fa0/4` on SW-Students), and on `CORE-HQ` the ports `Fa0/5`, `Fa0/6` and `Fa0/7` must be `connected` in VLAN 40.

## 4. Addressing plan (this is only for reference nothing to be done in this step: jump to step 5 for next task)

Use this plan exactly; it is also your VLSM evidence. This step is a reference only. Nothing is configured here; the addresses are entered in Steps 5 to 10.

### HQ VLANs and servers

| VLAN | Name | Network | Mask | Gateway |
|---:|---|---|---|---|
| 10 | ADMIN | `10.10.10.0/27` | `255.255.255.224` | `10.10.10.1` |
| 20 | FACULTY | `10.10.20.0/26` | `255.255.255.192` | `10.10.20.1` |
| 30 | STUDENTS | `10.10.30.0/25` | `255.255.255.128` | `10.10.30.1` |
| 40 | SERVERS | `10.10.40.0/28` | `255.255.255.240` | `10.10.40.1` |
| 99 | MANAGEMENT | `10.10.99.0/28` | `255.255.255.240` | `10.10.99.1` |

### Branch LANs

| Site | Network | Router gateway | Switch management IP |
|---|---|---|---|
| Engineering | `10.10.50.0/28` | `10.10.50.1` | `10.10.50.2` |
| Administration | `10.10.60.0/28` | `10.10.60.1` | `10.10.60.2` |
| Hostel | `10.10.70.0/28` | `10.10.70.1` | `10.10.70.2` |

### Router transit and WAN links

| Link | First address | Second address | Mask |
|---|---|---|---|
| `CORE-HQ` to `R-HQ` | Core `10.10.254.17` | R-HQ `10.10.254.18` | `255.255.255.252` |
| HQ to Engineering | HQ `10.10.254.1` | Engineering `10.10.254.2` | `255.255.255.252` |
| Engineering to Administration | Engineering `10.10.254.5` | Administration `10.10.254.6` | `255.255.255.252` |
| Administration to Hostel | Administration `10.10.254.9` | Hostel `10.10.254.10` | `255.255.255.252` |
| Hostel to HQ | Hostel `10.10.254.13` | HQ `10.10.254.14` | `255.255.255.252` |

### Static server addresses

| Server | IP address | Mask | Gateway | DNS |
|---|---|---|---|---|
| `Server-DNS-WEB` | `10.10.40.2` | `255.255.255.240` | `10.10.40.1` | `10.10.40.2` |
| `Server-FTP` | `10.10.40.3` | `255.255.255.240` | `10.10.40.1` | `10.10.40.2` |
| `Server-MAIL` | `10.10.40.4` | `255.255.255.240` | `10.10.40.1` | `10.10.40.2` |

### Interface-to-address map (which port gets which address)

Use this with the cabling tables in Step 3. Every routed or WAN interface, the port it is cabled to, and the address it carries:

| Device | Interface | IP address / mask | Cabled to |
|---|---|---|---|
| `CORE-HQ` | `Fa0/1` (routed, `no switchport`) | `10.10.254.17/30` | `R-HQ Fa0/0` |
| `CORE-HQ` | `Vlan10` | `10.10.10.1/27` | (SVI, ADMIN gateway) |
| `CORE-HQ` | `Vlan20` | `10.10.20.1/26` | (SVI, FACULTY gateway) |
| `CORE-HQ` | `Vlan30` | `10.10.30.1/25` | (SVI, STUDENTS gateway) |
| `CORE-HQ` | `Vlan40` | `10.10.40.1/28` | (SVI, SERVERS gateway) |
| `CORE-HQ` | `Vlan99` | `10.10.99.1/28` | (SVI, MANAGEMENT gateway) |
| `R-HQ` | `Fa0/0` | `10.10.254.18/30` | `CORE-HQ Fa0/1` |
| `R-HQ` | `Serial0/3/0` (DCE) | `10.10.254.1/30` | `R-Engineering Serial0/3/0` |
| `R-HQ` | `Serial0/3/1` | `10.10.254.14/30` | `R-Hostel Serial0/3/1` |
| `R-Engineering` | `Fa0/0` | `10.10.50.1/28` | `SW-Engineering Fa0/24` |
| `R-Engineering` | `Serial0/3/0` | `10.10.254.2/30` | `R-HQ Serial0/3/0` |
| `R-Engineering` | `Serial0/3/1` (DCE) | `10.10.254.5/30` | `R-Administration Serial0/3/0` |
| `R-Administration` | `Fa0/0` | `10.10.60.1/28` | `SW-Administration Fa0/24` |
| `R-Administration` | `Serial0/3/0` | `10.10.254.6/30` | `R-Engineering Serial0/3/1` |
| `R-Administration` | `Serial0/3/1` (DCE) | `10.10.254.9/30` | `R-Hostel Serial0/3/0` |
| `R-Hostel` | `Fa0/0` | `10.10.70.1/28` | `SW-Hostel Fa0/24` |
| `R-Hostel` | `Serial0/3/0` | `10.10.254.10/30` | `R-Administration Serial0/3/1` |
| `R-Hostel` | `Serial0/3/1` (DCE) | `10.10.254.13/30` | `R-HQ Serial0/3/1` |
| `SW-Admin` | `Vlan99` | `10.10.99.2/28` | (management) |
| `SW-Faculty` | `Vlan99` | `10.10.99.3/28` | (management) |
| `SW-Students` | `Vlan99` | `10.10.99.4/28` | (management) |
| `SW-Engineering` | `Vlan1` | `10.10.50.2/28` | (management) |
| `SW-Administration` | `Vlan1` | `10.10.60.2/28` | (management) |
| `SW-Hostel` | `Vlan1` | `10.10.70.2/28` | (management) |
| Servers | `Fa0` | `10.10.40.2` / `.3` / `.4` (/28) | `CORE-HQ Fa0/5`, `Fa0/6`, `Fa0/7` |
| All PCs | `Fa0` | DHCP | switch ports in Step 3 |

Routers and switches have no Desktop tab; they are configured on the **CLI** tab (Steps 5 to 8). Servers are configured through **Desktop > IP Configuration** (Step 9), and PCs use DHCP (Step 10).

## 5. Configure the HQ multilayer switch

Click `CORE-HQ` > **CLI**. When asked `Continue with configuration dialog?`, type `no`, then press Enter. Paste or type this configuration.

**Fix compared with the original guide:** the 3560 supports both ISL and 802.1Q, so `switchport mode trunk` is rejected until the encapsulation is chosen. The block below adds `switchport trunk encapsulation dot1q` for ports Fa0/2 to Fa0/4. Without it the trunks stay as ordinary access ports in VLAN 1 and VLANs 10, 20, 30 and 99 never come up.

```cisco_ios
enable
configure terminal
hostname CORE-HQ
no ip domain-lookup
ip routing

vlan 10
 name ADMIN
vlan 20
 name FACULTY
vlan 30
 name STUDENTS
vlan 40
 name SERVERS
vlan 99
 name MANAGEMENT

interface vlan 10
 ip address 10.10.10.1 255.255.255.224
 no shutdown
interface vlan 20
 ip address 10.10.20.1 255.255.255.192
 no shutdown
interface vlan 30
 ip address 10.10.30.1 255.255.255.128
 no shutdown
interface vlan 40
 ip address 10.10.40.1 255.255.255.240
 no shutdown
interface vlan 99
 ip address 10.10.99.1 255.255.255.240
 no shutdown

interface fa0/1
 no switchport
 ip address 10.10.254.17 255.255.255.252
 no shutdown

interface range fa0/2 - 4
 switchport trunk encapsulation dot1q

interface fa0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,99
interface fa0/3
 switchport mode trunk
 switchport trunk allowed vlan 20,99
interface fa0/4
 switchport mode trunk
 switchport trunk allowed vlan 30,99

interface range fa0/5 - 7
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast

ip dhcp excluded-address 10.10.10.1 10.10.10.9
ip dhcp excluded-address 10.10.20.1 10.10.20.9
ip dhcp excluded-address 10.10.30.1 10.10.30.9
ip dhcp pool HQ_ADMIN_POOL
 network 10.10.10.0 255.255.255.224
 default-router 10.10.10.1
 dns-server 10.10.40.2
 domain-name university.local
ip dhcp pool HQ_FACULTY_POOL
 network 10.10.20.0 255.255.255.192
 default-router 10.10.20.1
 dns-server 10.10.40.2
 domain-name university.local
ip dhcp pool HQ_STUDENT_POOL
 network 10.10.30.0 255.255.255.128
 default-router 10.10.30.1
 dns-server 10.10.40.2
 domain-name university.local

router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface fa0/1
 network 10.10.10.0 0.0.0.31 area 0
 network 10.10.20.0 0.0.0.63 area 0
 network 10.10.30.0 0.0.0.127 area 0
 network 10.10.40.0 0.0.0.15 area 0
 network 10.10.99.0 0.0.0.15 area 0
 network 10.10.254.16 0.0.0.3 area 0

ip domain-name university.local
username netadmin privilege 15 secret Cisco@123
crypto key generate rsa
1024
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
end
copy running-config startup-config
```

Check with:

```cisco_ios
show vlan brief
show interfaces trunk
show ip interface brief
show ip ospf neighbor
show ip route
show cdp neighbors
show interfaces status
```

What to expect at this stage:

- `show interfaces trunk` lists `Fa0/2`, `Fa0/3` and `Fa0/4` as `trunking` with 802.1q and allowed VLANs `10,99`, `20,99` and `30,99`. The "forwarding state" column stays `none` until the access switches in Step 6 are configured.
- `show interfaces status` shows `Fa0/1` as `routed`, `Fa0/5` to `Fa0/7` as `connected` in VLAN 40.
- `show ip ospf neighbor` is empty until `R-HQ` is configured (Step 7). Once both sides are up it shows `2.2.2.2` (R-HQ) as `FULL` on `FastEthernet0/1`.
- `show ip dhcp pool` should show the three HQ pools with 30, 62 and 126 addresses.
- The portfast warnings while entering the configuration are harmless.

## 6. Configure HQ access switches

The three HQ access-switch configurations are nearly identical. Enter commands using the **CLI** tab. The port-security blocks protect PC ports; the unused ports are shut down.

The commands in this step are unchanged. Make sure the cabling matches Step 3 first: the uplink on `Fa0/24`, the PCs on the ports listed in Step 3.4. Packet Tracer can retain a sticky MAC from a cable you later moved.

### SW-Admin

```cisco_ios
enable
configure terminal
hostname SW-Admin
no ip domain-lookup
vlan 10
 name ADMIN
vlan 99
 name MANAGEMENT
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,99
interface range fa0/1 - 3
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
interface vlan 99
 ip address 10.10.99.2 255.255.255.240
 no shutdown
ip default-gateway 10.10.99.1
interface range fa0/4 - 23
 switchport mode access
 switchport access vlan 99
 shutdown
end
copy running-config startup-config
```

### SW-Faculty

```cisco_ios
enable
configure terminal
hostname SW-Faculty
no ip domain-lookup
vlan 20
 name FACULTY
vlan 99
 name MANAGEMENT
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 20,99
interface range fa0/1 - 3
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
interface vlan 99
 ip address 10.10.99.3 255.255.255.240
 no shutdown
ip default-gateway 10.10.99.1
interface range fa0/4 - 23
 switchport mode access
 switchport access vlan 99
 shutdown
end
copy running-config startup-config
```

### SW-Students

```cisco_ios
enable
configure terminal
hostname SW-Students
no ip domain-lookup
vlan 30
 name STUDENTS
vlan 99
 name MANAGEMENT
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 30,99
interface range fa0/1 - 4
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
interface vlan 99
 ip address 10.10.99.4 255.255.255.240
 no shutdown
ip default-gateway 10.10.99.1
interface range fa0/5 - 23
 switchport mode access
 switchport access vlan 99
 shutdown
end
copy running-config startup-config
```

Verify each switch with `show interfaces status`. The PCs must be `connected` on `Fa0/1` to `Fa0/3` (`Fa0/1` to `Fa0/4` on SW-Students), `Fa0/24` must be `trunk`, and every other port must be `disabled`.

**Only if you moved a PC cable after pasting the configuration**, clear the old sticky MACs on the affected ports (use `fa0/1 - 4` on SW-Students):

```cisco_ios
enable
configure terminal
interface range fa0/1 - 3
 shutdown
 no switchport port-security mac-address sticky
 no switchport port-security
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
end
copy running-config startup-config
```

For evidence, generate traffic from each PC, then run:

```cisco_ios
show port-security
show port-security interface fa0/1
show mac address-table
```

## 7. Configure all routers

On every router, click **CLI**, type `no` if the setup dialog appears, then enter the matching configuration below. The shared CHAP password is `Uni@123`. The `username` must exactly match the peer router hostname.

The serial interfaces in this guide are `Serial0/3/0` and `Serial0/3/1`. If `show ip interface brief` on your router lists different serial names, substitute them everywhere below, including the `no passive-interface` lines.

### R-HQ

```cisco_ios
enable
configure terminal
hostname R-HQ
no ip domain-lookup
username R-Engineering password Uni@123
username R-Hostel password Uni@123
interface fa0/0
 ip address 10.10.254.18 255.255.255.252
 no shutdown
interface serial0/3/0
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.1 255.255.255.252
 clock rate 64000
 no shutdown
interface serial0/3/1
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.14 255.255.255.252
 no shutdown
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface fa0/0
 no passive-interface serial0/3/0
 no passive-interface serial0/3/1
 network 10.10.254.0 0.0.0.3 area 0
 network 10.10.254.12 0.0.0.3 area 0
 network 10.10.254.16 0.0.0.3 area 0
ip domain-name university.local
username netadmin privilege 15 secret Cisco@123
crypto key generate rsa
1024
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
end
copy running-config startup-config
```

### R-Engineering

```cisco_ios
enable
configure terminal
hostname R-Engineering
no ip domain-lookup
username R-HQ password Uni@123
username R-Administration password Uni@123
interface fa0/0
 ip address 10.10.50.1 255.255.255.240
 no shutdown
interface serial0/3/0
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.2 255.255.255.252
 no shutdown
interface serial0/3/1
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.5 255.255.255.252
 clock rate 64000
 no shutdown
ip dhcp excluded-address 10.10.50.1 10.10.50.3
ip dhcp pool ENGINEERING_POOL
 network 10.10.50.0 255.255.255.240
 default-router 10.10.50.1
 dns-server 10.10.40.2
 domain-name university.local
router ospf 1
 router-id 3.3.3.3
 passive-interface default
 no passive-interface serial0/3/0
 no passive-interface serial0/3/1
 network 10.10.50.0 0.0.0.15 area 0
 network 10.10.254.0 0.0.0.3 area 0
 network 10.10.254.4 0.0.0.3 area 0
ip domain-name university.local
username netadmin privilege 15 secret Cisco@123
crypto key generate rsa
1024
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
end
copy running-config startup-config
```

### R-Administration

```cisco_ios
enable
configure terminal
hostname R-Administration
no ip domain-lookup
username R-Engineering password Uni@123
username R-Hostel password Uni@123
interface fa0/0
 ip address 10.10.60.1 255.255.255.240
 no shutdown
interface serial0/3/0
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.6 255.255.255.252
 no shutdown
interface serial0/3/1
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.9 255.255.255.252
 clock rate 64000
 no shutdown
ip dhcp excluded-address 10.10.60.1 10.10.60.3
ip dhcp pool ADMINISTRATION_POOL
 network 10.10.60.0 255.255.255.240
 default-router 10.10.60.1
 dns-server 10.10.40.2
 domain-name university.local
router ospf 1
 router-id 4.4.4.4
 passive-interface default
 no passive-interface serial0/3/0
 no passive-interface serial0/3/1
 network 10.10.60.0 0.0.0.15 area 0
 network 10.10.254.4 0.0.0.3 area 0
 network 10.10.254.8 0.0.0.3 area 0
ip domain-name university.local
username netadmin privilege 15 secret Cisco@123
crypto key generate rsa
1024
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
end
copy running-config startup-config
```

### R-Hostel

```cisco_ios
enable
configure terminal
hostname R-Hostel
no ip domain-lookup
username R-Administration password Uni@123
username R-HQ password Uni@123
interface fa0/0
 ip address 10.10.70.1 255.255.255.240
 no shutdown
interface serial0/3/0
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.10 255.255.255.252
 no shutdown
interface serial0/3/1
 encapsulation ppp
 ppp authentication chap
 ip address 10.10.254.13 255.255.255.252
 clock rate 64000
 no shutdown
ip dhcp excluded-address 10.10.70.1 10.10.70.3
ip dhcp pool HOSTEL_POOL
 network 10.10.70.0 255.255.255.240
 default-router 10.10.70.1
 dns-server 10.10.40.2
 domain-name university.local
router ospf 1
 router-id 5.5.5.5
 passive-interface default
 no passive-interface serial0/3/0
 no passive-interface serial0/3/1
 network 10.10.70.0 0.0.0.15 area 0
 network 10.10.254.8 0.0.0.3 area 0
 network 10.10.254.12 0.0.0.3 area 0
ip domain-name university.local
username netadmin privilege 15 secret Cisco@123
crypto key generate rsa
1024
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
end
copy running-config startup-config
```

### Router verification

On each router run:

```cisco_ios
show ip interface brief
show interfaces serial 0/3/0
show ip ospf neighbor
show ip route
show running-config | section router ospf
```

All WAN serial links should be `up/up`, and `show interfaces serial` must say `Encapsulation PPP`. Wait about 40 seconds after all four routers are configured, then check the neighbour tables:

- `R-HQ` should list three neighbours: `3.3.3.3` (Serial0/3/0), `5.5.5.5` (Serial0/3/1) and `1.1.1.1` (FastEthernet0/0, CORE-HQ).
- `R-Engineering`, `R-Administration` and `R-Hostel` each list two neighbours.
- `show ip route` on any branch router should include `O` routes for 10.10.10.0, 10.10.20.0, 10.10.30.0, 10.10.40.0 and 10.10.99.0 as well as the other branch LANs.

If a PPP link stays down, verify both CHAP usernames/passwords and confirm that hostname spelling matches the peer username entry. If `R-HQ` shows no `1.1.1.1` neighbour, check `show cdp neighbors` on `CORE-HQ`: `R-HQ` must be on `Fa0/1` (not `Fa0/7`).

## 8. Configure branch switches

Each branch switch keeps its PCs in VLAN 1 because the branch has one LAN. Use the matching configuration. The router provides DHCP; the switch IP is only for management.

Before pasting, run `show cdp neighbors` on each branch switch and confirm the branch router is on `Fa0/24`. This configuration shuts down `Fa0/4` to `Fa0/23`, so a router cabled to any of those ports would be cut off.

### SW-Engineering

```cisco_ios
enable
configure terminal
hostname SW-Engineering
no ip domain-lookup
interface vlan 1
 ip address 10.10.50.2 255.255.255.240
 no shutdown
ip default-gateway 10.10.50.1
interface range fa0/1 - 3
 switchport mode access
 spanning-tree portfast
interface range fa0/4 - 23
 shutdown
end
copy running-config startup-config
```

### SW-Administration

```cisco_ios
enable
configure terminal
hostname SW-Administration
no ip domain-lookup
interface vlan 1
 ip address 10.10.60.2 255.255.255.240
 no shutdown
ip default-gateway 10.10.60.1
interface range fa0/1 - 3
 switchport mode access
 spanning-tree portfast
interface range fa0/4 - 23
 shutdown
end
copy running-config startup-config
```

### SW-Hostel

```cisco_ios
enable
configure terminal
hostname SW-Hostel
no ip domain-lookup
interface vlan 1
 ip address 10.10.70.2 255.255.255.240
 no shutdown
ip default-gateway 10.10.70.1
interface range fa0/1 - 3
 switchport mode access
 spanning-tree portfast
interface range fa0/4 - 23
 shutdown
end
copy running-config startup-config
```

## 9. Configure servers through Desktop -> IP Configuration

For each server, click the server, then click **Desktop** > **IP Configuration** > select **Static**. Fill the values from the table below exactly.

| Server | IP Address | Subnet Mask | Default Gateway | DNS Server |
|---|---|---|---|---|
| Server-DNS-WEB | `10.10.40.2` | `255.255.255.240` | `10.10.40.1` | `10.10.40.2` |
| Server-FTP | `10.10.40.3` | `255.255.255.240` | `10.10.40.1` | `10.10.40.2` |
| Server-MAIL | `10.10.40.4` | `255.255.255.240` | `10.10.40.1` | `10.10.40.2` |

### DNS and HTTP server settings

1. Click `Server-DNS-WEB` > **Services** > **HTTP** > switch **HTTP** to **On**.
2. Optionally edit the HTML page to show `Welcome to the University Network Portal`.
3. Click **Services** > **DNS** > switch **DNS** to **On**.
4. Add these A records, pressing **Add** after every row:

| Name | Address |
|---|---|
| `university.local` | `10.10.40.2` |
| `www.university.local` | `10.10.40.2` |
| `ftp.university.local` | `10.10.40.3` |
| `mail.university.local` | `10.10.40.4` |

### FTP server settings

1. Click `Server-FTP` > **Services** > **FTP**.
2. Switch the FTP service to **On**.
3. Under **User Setup**, configure the user accounts with the appropriate permissions:

| Username | Password | Permissions to Check | Purpose / Security Rationale |
|---|---|---|---|
| `student` | `cisco123` | **Read**, **List** *(Optionally check **Write**)* | **Least Privilege Access:** Allows students to view files (`dir`) and download course resources (`get`). Leave **Delete** and **Rename** unchecked to prevent accidental or malicious file tampering. *(Check **Write** if students must upload assignment files via `put`)*. |
| `admin` | `cisco123` | **Read**, **Write**, **Delete**, **Rename**, **List** | Full administrative rights for server maintenance, uploading, and managing repository files. |

4. After typing the credentials and selecting the checkboxes for each account:
   - For `student`: Check **Read** and **List** (and **Write** if upload testing is needed), then click **Add**.
   - For `admin`: Check all 5 boxes (**Read**, **Write**, **Delete**, **Rename**, **List**), then click **Add**.
5. *(Optional)* Under the FTP file list, note the default Cisco sample files, or create a text file on a PC to test file transfers. (example file name: asa842-k8.bin)
6. **Verify FTP access from a Student PC (`Student-PC1` > Desktop > Command Prompt):** (if you're checking with PC make sure to first goto Desktop -> IP Configuration and select DHCP IP Configuration instead of Static).

```text
ftp ftp.university.local
```
*(or `ftp 10.10.40.3`)*

- Enter username: `student`
- Enter password: `cisco123`
- Type `dir` to list files (tests **List** permission).
- Type `get <filename>` to download a file (tests **Read** permission).

Enter the following commands after logging in.

```bash
dir
get asa842-k8.bin 
```

- Type `quit` to exit FTP session.

### Email server settings

1. Click `Server-MAIL` > **Services** > **EMAIL**.
2. Switch SMTP and POP3 services to **On**.
3. Enter domain name `university.local`.
4. Create two users: `admin` / `cisco123` and `student` / `cisco123`.
5. Click **Add** after creating each user.

## 10. Configure all PCs through Desktop -> IP Configuration

All PCs use DHCP. For every PC:

1. Click the PC.
2. Click **Desktop**.
3. Click **IP Configuration**.
4. Click the **DHCP** button.
5. Wait a few seconds. The IP address, subnet mask, default gateway, and DNS Server fields should populate.

Do this for all PCs. Verify by opening a PC and clicking **Desktop** > **Command Prompt** and entering:

```text
ipconfig /all
ping 10.10.40.2
ping university.local
```

The first cross-network ping may lose one packet while ARP resolves; repeat it before deciding it has failed.

## 11. Configure the ACL on CORE-HQ

This ACL lets Students use DNS, web, FTP, and mail services but blocks them from the Admin and Management VLANs. The final permit keeps the rest of the network functioning.

Click `CORE-HQ` > **CLI**, then enter:

```cisco_ios
enable
configure terminal
ip access-list extended STUDENT_POLICY
 remark Permit DNS from Student VLAN to DNS server
 permit udp 10.10.30.0 0.0.0.127 host 10.10.40.2 eq 53
 permit tcp 10.10.30.0 0.0.0.127 host 10.10.40.2 eq 53
 remark Permit web, FTP and email services
 permit tcp 10.10.30.0 0.0.0.127 host 10.10.40.2 eq 80
 permit tcp 10.10.30.0 0.0.0.127 host 10.10.40.3 eq 21
 permit tcp 10.10.30.0 0.0.0.127 host 10.10.40.3 eq 20
 permit tcp 10.10.30.0 0.0.0.127 host 10.10.40.4 eq 25
 permit tcp 10.10.30.0 0.0.0.127 host 10.10.40.4 eq 110
 deny ip 10.10.30.0 0.0.0.127 10.10.10.0 0.0.0.31
 deny ip 10.10.30.0 0.0.0.127 10.10.99.0 0.0.0.15
 permit ip 10.10.30.0 0.0.0.127 any
exit
interface vlan 30
 ip access-group STUDENT_POLICY in
end
copy running-config startup-config
```

Verify with:

```cisco_ios
show access-lists STUDENT_POLICY
show ip interface vlan 30
```

From a Student PC, `ping 10.10.10.x` should fail, while opening `http://university.local` should work.

## 12. Optional email-client demonstration

Use `Faculty-PC1` as the sender and `Student-PC1` as the receiver.

On each PC click **Desktop** > **Email** > **Configure**. Use:

| Field | Faculty-PC1 | Student-PC1 |
|---|---|---|
| Your Name | Faculty User | Student User |
| Email Address | `admin@university.local` | `student@university.local` |
| Incoming Mail Server | `mail.university.local` | `mail.university.local` |
| Outgoing Mail Server | `mail.university.local` | `mail.university.local` |
| Username | `admin` | `student` |
| Password | `cisco123` | `cisco123` |

Send a short message from the faculty PC to `student@university.local`, then click **Receive** on the student PC.

## 13. Demonstrate OSPF failover

First confirm normal connectivity from `Eng-PC1` to the web server:

```text
ping 10.10.40.2
```

On `R-HQ`, deliberately disable the direct HQ-to-Engineering WAN link:

```cisco_ios
enable
configure terminal
interface serial0/3/0
 shutdown
end
```

Wait for OSPF convergence. On `R-HQ`, run:

```cisco_ios
show ip route 10.10.50.0
```

The route should now use the path through `R-Hostel` and `R-Administration`. Repeat the ping from `Eng-PC1` to `10.10.40.2`; it should still work. Restore the direct link afterward:

```cisco_ios
enable
configure terminal
interface serial0/3/0
 no shutdown
end
copy running-config startup-config
```

## 14. Simulation-mode evidence

Switch to **Simulation** mode at the bottom right. Use **Edit Filters** and retain only the protocol you are explaining.

### DHCP DORA

1. On a Student PC, open **Desktop** > **Command Prompt**.
2. Enter `ipconfig /release`, then `ipconfig /renew`.
3. Step through the packets and identify DHCP Discover, Offer, Request, and Acknowledge.

### DNS and HTTP

1. Keep DNS and HTTP enabled in the filter.
2. On a Student PC, open **Desktop** > **Web Browser**.
3. Browse to `http://university.local`.
4. Step through DNS query/reply followed by TCP and HTTP traffic.

## 15. Troubleshooting checklist

| Symptom | Likely cause | Check/fix |
|---|---|---|
| `% Invalid interface type and number` on `interface serial0/0/0` | Wrong serial label; the module is in a different slot | `show ip interface brief`, then use the label shown (this guide uses `serial0/3/0` and `serial0/3/1`) |
| Serial link is `up/down` | DCE clock rate, PPP mismatch, or CHAP mismatch | `show controllers serial`, `show interfaces serial`, username/hostname/password |
| `clock rate` rejected ("applies only to DCE interfaces") | That cable is wired the other way round | Remove the command there and put it on the other end of the link |
| No OSPF neighbor | Wrong IP/mask, interface down, missing OSPF network, or cable on the wrong port | `show ip ospf neighbor`, `show ip interface brief`, `show cdp neighbors` |
| `R-HQ` has no OSPF neighbour `1.1.1.1` | R-HQ `Fa0/0` is cabled to a CORE-HQ port other than `Fa0/1` | `show cdp neighbors` on CORE-HQ; R-HQ must be on `Fa0/1`; redraw the cable (Step 3.3) |
| `show interfaces trunk` is empty on `CORE-HQ`, VLAN interfaces down | Missing `switchport trunk encapsulation dot1q` on the 3560 | Apply the Step 5 trunk block; `Fa0/2` to `Fa0/4` must show `trunking` |
| `Native VLAN mismatch` message in the CORE-HQ console | CORE-HQ is cabled to an access switch PC port instead of `Fa0/24` | `show cdp neighbors`; the neighbour port ID must be `Fa0/24` |
| Red marker on a PC cable | PC is on a port that Step 6/8 shuts down, or port security blocked it | `show interfaces status` on the switch; move the PC to `Fa0/1`-`Fa0/3` (`Fa0/1`-`Fa0/4` on SW-Students); reset sticky MACs if needed |
| `Fa0/7` on CORE-HQ down/down | R-HQ or Server-MAIL cabled to the wrong port, or device powered off | `show cdp neighbors`, hover over the cable, check the Physical tab power switch |
| PC has no DHCP address | VLAN/trunk or DHCP pool error | `show vlan brief`, `show interfaces trunk`, `show ip dhcp binding`, `show ip dhcp pool` |
| DHCP pool shows the wrong size | Mask typed incorrectly in the pool | `show ip dhcp pool`; HQ_ADMIN_POOL must be `255.255.255.224` (30 addresses) |
| DNS name cannot resolve | DNS record/service or PC DNS field wrong | Server DNS service, `ipconfig /all` |
| Web works by IP but not name | DNS record is wrong | Confirm `university.local -> 10.10.40.2` |
| Student cannot browse but should | ACL rule missing or applied in wrong direction | `show access-lists STUDENT_POLICY`, `show ip interface vlan 30` |
| Protected port stops working | Expected port-security restriction or sticky-MAC change | `show port-security interface fa0/1`; reconnect original PC or clear sticky configuration (Step 6) |
| FTP login succeeds but `dir` or `get` fails | Missing Read and List permissions for user `student` | Select `student` under `Server-FTP` > **Services** > **FTP**, check **Read** and **List**, then click **Save** |
| FTP connection refused from Student PC | FTP service is Off, wrong IP/DNS, or ACL issue | Check `Server-FTP` FTP service is **On**, verify DNS entry for `ftp.university.local`, and check ACL permits ports 20 & 21 |

## 16. Final exam checklist

- Save the `.pkt` file and a separate backup copy.
- Keep a printed or saved copy of this guide and the addressing table.
- Capture screenshots of the complete topology, VLSM table, interface-to-address map, CDP neighbour output, OSPF neighbour table, working PPP/CHAP link, DHCP assignment, DNS record, browser test, ACL denial, port-security status, and WAN failover.
- Practice the demo sequence until you can explain every line in the architecture diagram.

