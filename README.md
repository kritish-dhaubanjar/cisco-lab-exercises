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



