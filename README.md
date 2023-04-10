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
