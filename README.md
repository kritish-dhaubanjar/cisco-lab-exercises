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

- RIP Metric = Hop Count
- Maximum hop count is 15, paths more than 15 hops away are marked unreachable
- eg: 2 hop of 10 Mbps link each is preferred over 3 hop of 100 Mbps link each

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
- IS-IS Metric Cost is not automatically derived from interface bandwidth. All links have an equal cost by default
- Can manually configure the cost of links to manipulate path
- If you don't manually set the cost of links, then path with the lowest hop count will be used.

## EIGRP
```
R2(config)# router eigrp
R2(config-router)# no auto-summary
R2(config-router)# network 10.0.0.0 0.255.255.255
```

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
