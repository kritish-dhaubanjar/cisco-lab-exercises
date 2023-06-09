# Cisco Lab Exercises

## [Routers as DNS Clients](https://www.flackbox.com/dns-on-cisco-routers)
Note that routers cannot be DNS servers in Packet Tracer (it does not support the ‘ip dns server’ command) so we are using a Packet Tracer server device as the DNS server.

![image](https://user-images.githubusercontent.com/25634165/230908837-8e9b10ed-a9b8-4239-b55a-e80c883df4ed.png)

## Cisco Router and Switch Initial Configuration
```
SW1 (config)# hostname sw1-cardboard-box
sw1-cardboard-box (config)# interface FastEthernet 0/1
sw1-cardboard-box (config-if)# description Link to r1.cardboard.box
```
![image](https://user-images.githubusercontent.com/25634165/230958538-f54d6444-1af1-452a-8d9d-4a1e88e289f3.png)

## Speed and Duplex Settings
```
sw1-cardboard-box (config)# interface FastEthernet 0/1
sw1-cardboard-box (config-if)# duplex full
sw1-cardboard-box (config-if)# speed 100
```

## Verification
```
sw1-cardboard-box# show running-config
sw1-cardboard-box# show interfaces
sw1-cardboard-box# show ip interface brief
sw1-cardboard-box# show version
```

```
sw1-cardboard-box# show ip interface brief
```
- administratively down - shutdown
- down/down - Layer 1 Issue
- up/down - Layer 2 Issue or speed mismatch


## CDP (Cisco Discovery Protocol) & LLDP (Link Layer Discovery Protocol)
```
sw1-cardboard-box (config)# cdp run
sw1-cardboard-box (config)# no cdp run
sw1-cardboard-box (config-if)# no cdp enable
sw1-cardboard-box# show cdp
sw1-cardboard-box# show cdp neighbors
sw1-cardboard-box# show cdp neighbors detail
```

```
sw1-cardboard-box (config)# lldp run
sw1-cardboard-box (config)# no lldp run
sw1-cardboard-box (config-if)# no lldp transmit
sw1-cardboard-box (config-if)# no lldp receive
sw1-cardboard-box# show lldp
sw1-cardboard-box# show lldp neighbors
sw1-cardboard-box# show lldp neighbors detail
```

## [The Boot-up Process & Disaster Recovery with TFTP Download](https://www.cisco.com/en/US/docs/routers/access/800/850/software/configuration/guide/rommon.html)
```
R1# show flash:

System flash directory:
File  Length   Name/status
  3   5571584  pt1000-i-mz.122-28.bin
  2   28282    sigdef-category.xml
  1   227537   sigdef-default.xml
[5827403 bytes used, 58188981 available, 64016384 total]
63488K bytes of processor board System flash (Read/Write)

R1# copy flash tftp
Source filename []? pt1000-i-mz.122-28.bin
Address or name of remote host []? 10.10.10.254
Destination filename [pt1000-i-mz.122-28.bin]? pt1000-i-mz.122-28.bin

R1# delete flash:pt1000-i-mz.122-28.bin

R1# reload
...
Boot process failed...

The system is unable to boot automatically.  The BOOT
environment variable needs to be set to a bootable
image.

rommon 1 >
rommon 2 > IP_ADDRESS=10.10.10.1
rommon 3 > IP_SUBNET_MASK=255.255.255.0
rommon 4 > DEFAULT_GATEWAY=10.10.10.254
rommon 5 > TFTP_SERVER=10.10.10.254
rommon 6 > TFTP_FILE=pt1000-i-mz.122-28.bin
rommon 7 > tftpdnld
```

![image](https://user-images.githubusercontent.com/25634165/231179446-2f627621-b01f-43ad-88e8-a5338651cf3c.png)

### Factory Reset
```
write erase
```

### The Config Register (config-register/confreg)
```
confreg 0x2142
```
1. 0x2102: boot normally (default)
2. 0x2120: boot into rommon
3. 0x2142: ignore NVRAM contents (startup-config)

### Password Recovery
```
R1(config)# enable password <PASSWORD>
R1(config)# enable secret <PASSWORD>
R1(config)# do reload
```

Break the boot sequence [Ctrl+C] into rommon.
```
rommon 1 > confreg 0x2142
rommon 2 > reset
```

Preserve the startup-config
```
Router# copy startup-config running-config
Router# configure terminal
Router(config)# enable password <PASSWORD>
Router(config)# enable secret <PASSWORD>

Router(config)# config-register 0x2102
Router# copy running-config startup-config
Router# reload
```

## Summarisation, Longest Prefix Match, and Default Routes
For R1:
- Classful or Classless Boundaries
- Longest Prefix will be selected if more than one route (eg: 255.255.255.0 over 255.0.0.0)
- Load Balance between matching prefix
- Default Gateway or Gateway of Last Resort

```
ip route 0.0.0.0 0.0.0.0 <Gateway_X.X.X.X>
```

![image](https://user-images.githubusercontent.com/25634165/231238732-d2e6dc33-8891-4ade-b02f-b859f421fa4d.png)

## RIP
![image](https://user-images.githubusercontent.com/25634165/231259982-01ffc0f4-08cc-4681-9036-b83bd0764992.png)

```
R3(config)# router rip
R3(config-router)# no auto-summary
R3(config-router)# version 2
R3(config-router)# network 192.168.101.0
R3(config-router)# network 10.0.1.0
```

In R2
```
R2# debug ip routing
IP routing debugging is on

RT: SET_LAST_RDB for 192.168.101.0/24

    NEW rdb: via 10.0.1.1

RT: add 192.168.101.0/24 via 10.0.1.1, rip metric [120/1]

RT: NET-RED 192.168.101.0/24

R2#undebug all
```

Log:
```
show ip rip database
```

**Metric**
- RIP Metric = Hop Count
- Maximum hop count is 15, paths more than 15 hops away are marked unreachable
- eg: 2 hop of 10 Mbps link each is preferred over 3 hop of 100 Mbps link each

- It'll perform ECMP, for upto 4 paths by default

#### RIPv2 vs RIPv2
- RIPv1 doesn't send subnet mask information with routing updates, so VLSM is not supported
- RIPv1 updates are send every 30 seconds as broadcast traffic, RIPv2 uses multicast address 224.0.0.9
- RIPv1 doesn't support authentication, RIPv2 does.

#### Auto-Summary
- RIP will automatically summarize routes to classful boundary by default
- For example, 192.168.10.1/30 will be advertised as 192.168.10.0/24
- 172.16.10.1/30 will be advertised as 127.16.0.0/16

Hence:
```
R2(config)# router rip
R2(config-router)# no auto-summary
R2(config-router)# interface f1/0
R2(config-if)# ip summary-address rip 10.0.0.0 255.255.0.0
```

![image](https://user-images.githubusercontent.com/25634165/232237317-1dc3339c-1ccf-4e53-bdfc-caf74e49cd6c.png)

#### Default Route Injection
Default Internet Gateway: 203.0.113.2

```
R4(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
R4(config)# router rip
R4(config-router)# default-information originate
```

## Dynamic Routing Protocols
![image](https://user-images.githubusercontent.com/25634165/231264821-70813062-7506-4970-a7e3-119a5fc7cb15.png)

## OSPF
![image](https://user-images.githubusercontent.com/25634165/232225111-4e92fe00-9e5f-4396-9189-5ef1f835f9a9.png)

```
R2(config)# router ospf 1
R2(config-router)# network 10.0.0.0 0.255.255.255 area 1
R2(config-router)# end
R2# show ip ospf database 
            OSPF Router with ID (10.0.1.1) (Process ID 1)

                Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.0.2.2        10.0.2.2        1327        0x80000002 0x00136f 2
10.0.2.1        10.0.2.1        602         0x80000004 0x00a01f 4
10.0.1.1        10.0.1.1        566         0x80000004 0x0013b4 4
10.0.0.1        10.0.0.1        566         0x80000002 0x00eba2 2
R2#
```

If you don't enter a wildcard mask, the command **will not default** to using classful boundary. (Advertisement is similar to EIGRP)

- Link State Routing Protocol
- Supports large networks
- Fast Convergence Time
- Messages are sent using multicast
- Open Standard Protocol
- Uses Dijkstra's Shortest Path First algorithm

**Operations**
1. Discover Neighbours
2. Form Adjacencies
3. Flood Link State Database (LSDB)
4. Compute Shortest Path
5. Install best routes in routing table
6. Respond tp network changes

**OSPF Packet Types**
- **Hello:** A router will send out and listen for Hello packets when OSPF is enabled on an interface, and form adjacencies with other OSPF routers
- **DBD (DataBase Description)**: Adjacent routers will tell each other the networks they know about with the DBD packets
- **LSR (Link State Request)**: If a router is missing info about any of the networks in the received DBD, it will send the neighbour an LSR
- **LSA (Link State Advertisement)**: A routing update
- **LSU (Link State Update)**: Contains a list of LSA's which should be updated used during flooding
- **LSAck**: Receiving routers acknowledge LSAs

**OSPF Router ID** is similar to **EIGRP Router ID**

**Default Route Injection** is similar to **RIP Default Route Injection**

**Metric**
- OSPF Metric Cost = Interface Bandwidth (by default)
- Can manually configure the cost of links to manipulate path
- eg: 3 hop of 100 Mbps link each is preferred over 2 hop of 10 Mbps link each

**Areas**
- Every Router learns the full picture of the network, causing issues in large networks:
  - Too much router memory usage
  - Network changes cause all routers to re-converge wasting CPU resources
- Hierarchical design which segments large networks into smaller areas
- Each router maintains full info about its own area, but only summary info about other areas

![image](https://user-images.githubusercontent.com/25634165/232245353-2e740adf-b006-41f8-a130-f355ef4c76da.png)

- A two level hierarchy:
  - Transit area (backbone or area 0): Doesn't generally contain end users
  - Regular Areas (non-backbone areas): Uses to connect end users to the Transit area.
- Small networks don't require a hierarchical design, only Area 0 

![image](https://user-images.githubusercontent.com/25634165/232245522-e427ea7a-cfec-4be4-bb82-07d49b059508.png)

**Manual Summarization**
![image](https://user-images.githubusercontent.com/25634165/232245610-cacbd4e9-76f8-45fb-9cd0-0ea0dab47815.png)


## IS-IS
```
R2(config)# router isis
R2(config-router)# network 49.0001.0000.0000.0002.00
R2(config-router)# interface f0/0
R2(config-if)# ip router isis
R2(config-if)# interface f1/0
R2(config-if)# ip router isis
R2(config-if)# interface f2/0
R2(config-if)# ip router isis
R2(config-if)# interface f3/0
R2(config-if)# ip router isis
```

**Metric**
- IS-IS Metric Cost is not automatically derived from interface bandwidth. All links have an equal cost by default
- Can manually configure the cost of links to manipulate path
- If you don't manually set the cost of links, then path with the lowest hop count will be used.

## EIGRP
```
R2(config)# router eigrp 100
R2(config-router)# no auto-summary
R2(config-router)# network 10.0.0.0 0.255.255.255
```

`router eigrp <Autonomous System>`, meaning an independent administrative domain. EIGRP routers need to have the same AS number to peer with each other.

If you don't enter a wildcard mask, the command defaults to using classful boundary.
- 0.255.255.255 for a class A address
- 0.0.255.255 for a class B address
- 0.0.0.255 for a class C address

CAUTION:
![image](https://user-images.githubusercontent.com/25634165/232239736-a5093738-240a-48ab-b157-c7e0d7ddcca8.png)

```
R1(config-router)# network 10.0.0.0
```
Will only advertise networks
- 10.1.0.0/24
- 10.0.1.0/24
- 10.0.2.0/24
- 10.0.0.0/8 is NOT advertised

Alternatively:
```
R1(config-router)# network 10.1.0.0 0.0.0.255
R1(config-router)# network 10.0.1.0 0.0.0.255
R1(config-router)# network 10.0.2.0 0.0.0.255
```

```
R1(config-router)# network 10.1.0.1 0.0.0.0
R1(config-router)# network 10.0.1.1 0.0.0.0
R1(config-router)# network 10.0.2.1 0.0.0.0
```

- Advance Distance Vector Routing Protocol
- Supports large networks
- Fast Convergence Time
- Supports bounded updates, where network topology change updates are only sent to routes affected by the change
- Messages are sent using multicast
- Automatically perform equal cost load balancing on up to 4 paths by default, max 16 paths
- EIGRP is the only routing protocol capable of UnEqual Cost Multi Path. It must be manually configured to support this.

**EIGRP Router ID**
- EIGRP routes identify themselves using an EIGRP Router ID (form of an ip address)
- Default being the highest IP address of any loopback interfaces configures on router, ot the highest other IP address if loopback doesn't exist
- Loopback interface never go down, so the Router ID will not change
- Can manually specify Router ID
- Best practice is to use a loopback or manually set

```
R1(config)# router eigrp 100
R1(config-router)# eigrp router-id 2.2.2.2
```

2.2.2.2 is not an ip address, but just the format of ip address

**Metric**
- EIGRP Metric Cost = Uses bandwidth + delay of links to calculate metric
- (Load & reliability can also be considered but ignored by default)
- A fixed delay value is used based on interface bandwidth, this protocol doesn't dynamically measure current delay
- Can manually configure the delay of links to manipulate path

## Equal Cost Multi Path (ECMP)
- If multiple paths to a destination have an equal metric, the router will enter all of the paths into the routing table
- Load balance the outbound traffic to the destination over the different paths
- All IGP routing protocols will perform ECMP by default
- EIGRP is the only routing protocol capable of UnEqual Cost Multi Path. It must be manually configured to support this.

### Static Routes (Load Balancing)
```
R2(config)# ip route 10.0.1.0 255.255.255.0 10.1.1.2
R2(config)# ip route 10.0.1.0 255.255.255.0 10.1.3.2
```

## Administrative Distance
RIP: A>B>C>D hop count of 3, A>B>D has hop count of 2, so A>B>D is preferred
OSPF: A>B>C>D has cost of 60, A>B>D has a cost of 100, so A>B>C>D is preferred

RIP hop count cannot be compared with OSPF cost of 60. The comparison is meaningless because the metrics are completely different.

The Administrative Distance (AD) is a measure of how trusted the routing protocol is.

| Route Source | Default AD|
|-|-|
|Connected Interface|0|
|Static Route|1|
|External BGP|20|
|EIGRP|90|
|OSPF|110|
|IS-IS|115|
|RIP|120|

AD is used to choose between multiple paths learned via different routing protocols.

eg: [Administrative Distance/Metric] as [110/128]
```
R2#show ip route 
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 3 subnets
C       10.0.0.0 is directly connected, Serial2/0
C       10.0.1.0 is directly connected, Serial3/0
O       10.0.2.0 [110/128] via 10.0.1.2, 01:18:28, Serial3/0
```

#### Floating Static Routes - OSPF
We can change the AD of static route to make it act as the backup (rather than the preferred) route
eg: AD = 115
```
R2(config)# ip route 10.0.1.0 255.255.255.0 10.1.3.2 115
```

## Loopback Interface
- Logical interfaces
- Assign IP address to router or L3 Switch, which is not tied to a physical interface
- Cannot be physically in the same subnet as other devices, so they are usually assigned a /32 subnet mask to avoid wasting IP addresses
- Loopback is commonly used for traffic that terminates on router itself. eg: Management Traffic, VoIP, BGP etc
- Provides redundancy if there are multiple paths to the router
- Also used to identify the router in OSPF
- Multiple loopbacks can be configured, not common

![image](https://user-images.githubusercontent.com/25634165/232230508-683f3bd4-99f1-4caa-8bbe-895b73dc465a.png)

- Add interface loopback 0 with IP Address 192.168.1.1/32
- Advertise 192.168.1.1/32 in routing protocol
- R4 learns the 2 paths to 192.168.1.1
- PC can still connect to 192.168.1.1 even if either path goes down

```
R1(config)# interface loopback 0
R1(config-if)# ip address 192.168.1.1 255.255.255.255
R1(config-if)# do show running-config | section eigrp
R1(config-if)# router eigrp 100
R1(config-router)# network 192.168.1.1 0.0.0.0
```

## Adjacencies & Passive Interfaces
### Adjacencies
- IGP routing protocols are configured under global configuration mode and then enabled on individual interfaces
- When the routing protocol is enabled on an interface, the router will look for other devices on the link which are also running the routing protocol
- The router does this by sending out and listening for hello packets
- When a matching peer is found, the routers form an adjacency with each other
- Then they exchange routing information
- Modern routing protocol use multicast for the hello packets
- This is more efficient than broadcast which was used by earlier protocols
- Only routers which are running the same routing protocol will process the packet

![image](https://user-images.githubusercontent.com/25634165/232231110-2b4dacf0-eefa-499d-b26b-d458f8d54cda.png)

eg: Routing protocol is not enabled on FastEthernet2/0, we don't want to send internal network information to RC.
- It will form adjacencies with any routers running the same protocol on Loopback0, FastEthernet0/0 & 1/0
- It will not form an adjacency with RC
- *We'll use static routes for the extranet traffic with RC*
- RA & RB will not learn routes to 10.0.2.0/24

### Passive Interface
- Allow you to include an IP subnet in the routing protocol without sending updates out of the interface
- If FastEthernet2/0 is configured as a passive interface, RA & RB will learn routes to 10.0.2.0, but internal network information will not be sent to RC
- It is best practice to configure loopback interfaces as passive interfaces
- It is impossible to form adjacency on a loopback interface because they are not physical interfaces
- Making loopback passive means that it will be advertised by the routing protocol, but will not waste resources sending out or listening for hello packets

Passive Interface are used on:
- Loopback Interfaces
- Physical interfaces where we don't want to send routing information out but we do want our itnernal devices to know about the link

![image](https://user-images.githubusercontent.com/25634165/232230508-683f3bd4-99f1-4caa-8bbe-895b73dc465a.png)

```
R1(config)# interface loopback 0
R1(config-if)# ip address 192.168.1.1 255.255.255.255
R1(config) router rip
R1(config-router) no auto-summary
R1(config-router) version 2
R1(config-router) passive-interface loop0
R1(config-router) passive-interface f2/0
R1(config-router) network 192.168.1.1
R1(config-router) network 10.0.0.0
```

## VLAN
- Access Ports are configured with one specific VLAN
- Configuration is all on the switch, the end host is not VLAN aware
- All ports are in VLAN 1 by default

```
SW1(config)# vlan 10
SW1(config-vlan)# name VLAN_10

SW(config)# interface FastEthernet 0/1
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 10

SW(config)# interface range FastEthernet 0/3 - 5
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 10
```

### VLAN Trunk Ports
- Link between switches **Dot1Q Trunk**
- An access port carries traffic for one specific VLAN
- Dot1Q Trunks are configured on the links between switches where we need to carry traffic for multiple VLANs
- ISL (Inter-Switch Link) was a Cisco Proprietary trunking protocol which is now obsolete
- When the switch forwards traffic to another switch, it tags the layer 2 Dot1Q header with correct VLAN
- Receiving switch will only forward the traffic out ports that are in that VLAN
- The switch removes the Dot1Q tag from the Ethernet frame when it sends to the end host

![image](https://user-images.githubusercontent.com/25634165/232247460-d32784cc-7a81-42b0-91c0-a748389e4396.png)

```
SW(config)# interface FastEthernet 0/1
SW(config-if)# switchport trunk encapsulation dot1q
SW(config-if)# switchport mode trunk
SW(config-if)# switchport mode trunk native vlan 404
```

The Native VLAN is vlan 1, to assign to any traffic which comes in untagged on a trunk port, but for security, beset practice is to change it to an ununsed VLAN. The native VLAN must match on both switch.

**Allowed VLAN Config:**
```
SW1(config)# interface FastEthernet 0/1
SW1(config-if)# switchport trunk allowed vlan 10,30
```

**For Voice VLAN Configuration (Access for/instead Trunk)**
```
SW(config)# interface FastEthernet 0/1
SW(config-if)# switchport mode access
SW(config-if)# switchport access vlan 10
SW(config-if)# switchport voice vlan 20
```

#### Dynamic Trunking Protocol DTP
- Will form a trunk if the neighbour switch port is set to trunk or desirable.
- Trunk will not be formed if both sides are set to auto

```
SW(config)# interface FastEthernet 0/1
SW(config-if)# switchport mode dynamic auto
```

- Will form a trunk if the neighbour switch port is set to trunk, desirable or auto

```
SW(config)# interface FastEthernet 0/1
SW(config-if)# switchport mode dynamic desirable
```

- Disable DTP

```
SW(config)# interface FastEthernet 0/1
SW(config-if)# switchport nonnegotiate
```

#### VLAN Trunking Protocol VTP
- VTP allows to add, edit & delete VLANs on switches configured as VTP servers, and have other switches configured as VTP clients sync their VLAN database.
- You'll still need to perform port level VLAN config on the switches
- If both VTP & DTP are used, the VTP domain name has to match on neighbour switches for trunks to be formed by DTP

**VTP Modes**
- **VTP Server**: can add, edit or delete VLANs. A VTP server will sync its VLAN db from another server with a higher revision number
- **VTP Client**: cannot add, edit or delete VLANs. A VTP client will sync its VLAN db from another server with a higher revision number
- **VTP Transparent**: doesn't participate in the VTP domain. Doesn't advertise or learn VLAN information but will pass it on. Can add, eidt or delete VLANs in its own local VLAN db

![image](https://user-images.githubusercontent.com/25634165/232580557-ddd47a17-8ef1-47f1-86aa-ed5f6f13787c.png)

```
Server(config)# vtp domain CardboardBox
Server(config)# vtp mode server

Client(config)# vtp domain CardboardBox
Client(config)# vtp mode client
Client(config)# ! cannot add VLAN if VTP client

Transparent(config)# vtp mode transparent
Transparent(config)# vlan 20
Transparent(config-vlan)# name sales

Server# show vtp status
```

### Inter-VLAN Routing

1. Router with Separate Interfaces
![image](https://user-images.githubusercontent.com/25634165/232583231-59846fca-3315-4afa-9859-69e44451f81f.png)

2. Router on a Stick
![image](https://user-images.githubusercontent.com/25634165/232585020-c883a3d2-ac55-476a-b268-dd2aa4e58d79.png)

```
R1(config)# interface FastEthernet 0/1
R1(config-interface)# no ip address
R1(config-interface)# no shutdown

R1(config)# interface FastEthernet 0/1.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 10.10.10.1 255.255.255.0

R1(config)# interface FastEthernet 0/1.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 10.10.20.1 255.255.255.0

R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2

SW1(config)# interface FastEthernet 0/1
SW1(config-if)# switchport trunk encapsulation dot1q 20
SW1(config-if)# switchport mode trunk
```

3. Layer 3 Switch
![image](https://user-images.githubusercontent.com/25634165/232587439-5ee62495-e306-4bfa-988a-1d4c513a22d1.png)

```
SW1(config)# ip routing
SW1(config)# interface vlan 10
SW1(config-if)# ip address 10.10.10.1 255.255.255.0

SW1(config)# interface vlan 20
SW1(config-if)# ip address 10.10.20.1 255.255.255.0

SW1(config)# interface FastEthernet 0/1
SW1(config-if)# no switchport
SW1(config-if)# ip address 10.10.100.1 255.255.255.0
SW1(config-if)# ip route 0.0.0.0 0.0.0.0 10.10.100.2
```


### DHCP (Dynamic Host Configuration Protocol)
1. Cisco DHCP Server Configuration

```
R1(config)# ip dhcp excluded-address 10.10.10.1 10.10.10.10
R1(config)# ip dhcp pool 10.10.10.0_Clients
R1(dhcp-config)# network 10.10.10.0 255.255.255.0
R1(dhcp-config)# default-router 10.10.10.1
R1(dhcp-config)# dns-server 10.10.20.10

R1# show ip dhcp pool
R1# show ip dhcp binding
```

2. External DHCP Server Configuration

![image](https://user-images.githubusercontent.com/25634165/233151884-5d850b50-c729-4422-bbec-0f65717ec0e8.png)

```
R1(config)# interface f0/1
R1(config-if)# ip helper-address 10.10.20.10
```

#### Cisco Router as a DHCP Client
```
R1(config)# interface f0/0
R1(config-if)# ip address dhcp
R1(config-if)# no shutdown

R1# show dhcp lease
```

### NAT (Network Address Translation)
- **Static NAT**: permanent 1-1 mapping usually between a public and private IP address. Used for servers which must accept incoming connections.

- **Dynamic NAT**: uses a pool of public addresses which are given out on an as needed first come first served basis. Usually for internal hosts which needs to connect to the internet but don ot accept incoming connections.

- **PAT (Port Address Translation)**: allows same public IP address to be reused.

#### Static NAT
![image](https://user-images.githubusercontent.com/25634165/233157363-e391d507-f6e2-4594-a1e9-f9c9e103d0b0.png)

- f0/0 is outside interface
- f1/0 is inside interface
- We've bought the range of public ip address 203.0.113.0/28 from ISP
- 203.0.113.2 is used on f0/0 (R1)
- 203.0.113.1 is the default gateway address (SP1)
- 203.0.113.3 - 203.0.113.14 remain available
- We need to assign a fixed public IP (203.0.113.3) to accept incoming connections.
- A static NAT translation is required to translate public IP address 203.0.113.3 on f0/0 to 10.0.1.10 on F1/0 for incoming connections
- The translation is bidirectional so will also translate 10.0.1.10 to 203.0.113.3 for outbound traffic from the server

```
R1(config)# int f0/0
R1(config-if)# ip nat outside

R1(config)# int f1/0
R1(config-if)# ip nat inside

R1(config)# ip nat inside source static 10.0.1.10 203.0.113.3

R1# show ip nat translation
```

#### NAT Definations (from R1's perspective)
- Inside Local Address: IP address actually configured on the inside host's OS (10.0.1.10)
- Inside Global Address: NAT'd address of the inside host as it will be reached by the outside network (203.0.113.3)
- Outside Local Adddress: IP address of outside host as it appears to the inside network (203.0.113.20)
- Outside Global address: IP address assigned to the host on the outside network by the host's owner (203.0.113.20)

**Outside Local Adddress vs Outside global address**
![image](https://user-images.githubusercontent.com/25634165/233162156-3136fec7-14f4-489f-b799-bd86a8c983cd.png)

![image](https://user-images.githubusercontent.com/25634165/233162248-dcd37dd8-68cc-4e5b-a968-4e0e72bc0388.png)

#### Dynamic NAT
- 203.0.113.4 - 203.0.113.14 remain available
- The hosts in 10.0.2.0/24 don't need to accept incoming connections so they don't need a fixed public IP address with static NAT.
- They do need outbound connectivity to internet so need to be translated to public IP address
- The 1st host to send traffic out will be translated to 203.0.113.4, the second host to 203.0.113.5 etc (like FIFO) up to end of the pool (203.0.113.14)
- With Standard dynamic NAT, you need a public IP address for every host which needs to communicate with the outside
- If you've 20 hosts, you need 30 public IP address
- The hosts would have to await for existing connections to be torn down and the translations to be released back into the pool when they timeout

```
R1(config)# int f0/0
R1(config-if)# ip nat outside

R1(config)# int f2/0
R1(config-if)# ip nat inside

R1(config)#! Configure the pool of global addresses
R1(config)# ip nat pool CarboardBox 203.0.113.4 203.0.113.14 netmask 255.255.255.240

R1(config)#! Create an access list which references the internal IP addresses we want to translate
R1(config)# access-list 1 permit 10.0.2.0 0.0.0.255

R1(config)#! Associate the access list with the NAT pool to complete the configuration
R1(config)# ip nat inside source list 1 pool CardboardBox

R1# show ip nat translation
R1# show ip nat statistics
R1# clear ip nat translation
R1# clear ip nat translation *
```

#### PAT (Port Address Translation)
- Dynamic NAT with Overload (uses PAT): When Public IP runs out from pool, use the same public IP with different port
- When return traffic is sent back, the router checks the destination port number to see which host to forward it to

![image](https://user-images.githubusercontent.com/25634165/233169075-639a5000-5c21-4e89-b412-fcfb9df2b81c.png)

![image](https://user-images.githubusercontent.com/25634165/233169101-d1098371-af60-425c-b236-aa1e3ccdf497.png)

```
R1(config)# int f0/0
R1(config-if)# ip nat outside

R1(config)# int f2/0
R1(config-if)# ip nat inside

R1(config)#! Configure the pool of global addresses
R1(config)# ip nat pool CarboardBox 203.0.113.4 203.0.113.14 netmask 255.255.255.240

R1(config)#! Create an access list which references the internal IP addresses we want to translate
R1(config)# access-list 1 permit 10.0.2.0 0.0.0.255

R1(config)#! Associate the access list with the NAT pool to complete the configuration
R1(config)# ip nat inside source list 1 pool CardboardBox overload
```

##### PAT with Single IP Address
```
R1(config)# int f0/0
R1(config-if)# ip address dhcp
R1(config-if)# ip nat outside

R1(config)# int f2/0
R1(config-if)# ip nat inside

R1(config)#! Create an access list which references the internal IP addresses we want to translate
R1(config)# access-list 1 permit 10.0.2.0 0.0.0.255

R1(config)#! Associate the access list with the outside interface to complete the configuration
R1(config)# ip nat inside source list 1 interface f0/0 overload
```
