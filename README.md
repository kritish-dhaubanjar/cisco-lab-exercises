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
**Metric**
- OSPF Metric Cost = Interface Bandwidth (by default)
- Can manually configure the cost of links to manipulate path
- eg: 3 hop of 100 Mbps link each is preferred over 2 hop of 10 Mbps link each

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
- Automatically poerform equal cost load balancing on up to 4 paths by default, max 16 paths
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
