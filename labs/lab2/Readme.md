# Лабораторная работа 2 "Underlay. OSPF"
## Схема сети:

![alt text](Net_Scheme_Lab1.png)

## Таблица адресов:
| Подсеть ipv4 | Device/Port|    Описание   |
|--------------|:----------:| -----------------:|
| 192.168.10.1/32  | Spine-1-1/Lo1 |     Loopback1     |
| 192.168.10.2/32  | Spine-1-2/Lo1 |     Loopback1     |
| 192.168.12.1/32  |  Leaf-1-1/Lo1 |     Loopback1     |
| 192.168.13.1/32  |  Leaf-1-1/Lo2 |     Loopback2     |
| 192.168.12.2/32  |  Leaf-1-2/Lo1 |     Loopback1     |
| 192.168.13.2/32  |  Leaf-1-2/Lo2 |     Loopback2     |
| 192.168.12.3/32  |  Leaf-1-3/Lo1 |     Loopback1     |
| 192.168.13.3/32  |  Leaf-1-3/Lo2 |     Loopback2     |
| 192.168.14.0/31  |  Spine-1-1 Eth1 |     P2P Spine 1-1 to Leaf 1-1    |
| 192.168.14.1/31  |  Leaf-1-1 Eth1 |     P2P Spine 1-1 to Leaf 1-1    |
| 192.168.14.2/31  |  Spine-1-1 Eth2 |     P2P Spine 1-1 to Leaf 1-2    |
| 192.168.14.3/31  |  Leaf-1-2 Eth1 |     P2P Spine 1-1 to Leaf 1-2    |
| 192.168.14.4/31  |  Spine-1-1 Eth3 |     P2P Spine 1-1 to Leaf 1-3    |
| 192.168.14.5/31  |  Leaf-1-3 Eth1 |     P2P Spine 1-1 to Leaf 1-3    |
| 192.168.14.6/31  |  Spine-1-2 Eth1 |     P2P Spine 1-2 to Leaf 1-1    |
| 192.168.14.7/31  |  Leaf-1-1 Eth2 |     P2P Spine 1-2 to Leaf 1-1    |
| 192.168.14.8/31  |  Spine-1-2 Eth2 |     P2P Spine 1-2 to Leaf 1-2    |
| 192.168.14.9/31  |  Leaf-1-2 Eth2 |     P2P Spine 1-2 to Leaf 1-2    |
| 192.168.14.10/31  |  Spine-1-2 Eth3 |     P2P Spine 1-2 to Leaf 1-3    |
| 192.168.14.11/31  |  Leaf-1-3 Eth2 |     P2P Spine 1-2 to Leaf 1-3    |

## Настройки коммутаторов:
### Типовая конфигурация процесса OSPF:
```console
router ospf 10
   router-id <IP Loopback1>>
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
```
### Типова конфигурация порта между оммутаторами:
```console
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
```
### SPINE-1-1:
```console
!
hostname SPINE-1-1
!
interface Ethernet1
   no switchport
   ip address 192.168.14.0/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 IQqLTq4evBW2IbLOS7uR2Q==
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.14.2/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 192.168.14.4/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 192.168.10.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 10
   router-id 192.168.10.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
```
### SPINE-1-2:
```console
!
hostname SPINE-1-2
!
interface Ethernet1
   no switchport
   ip address 192.168.14.6/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 IQqLTq4evBW2IbLOS7uR2Q==
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.14.8/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 192.168.14.10/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 192.168.10.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 10
   router-id 192.168.10.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
```
### LEAF-1-1:
```console
!
hostname LEAF-1-1
!
interface Ethernet1
   description SPINE-1-1 Eth1
   no switchport
   ip address 192.168.14.1/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 IQqLTq4evBW2IbLOS7uR2Q==
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description SPINE-1-2 Eth1
   no switchport
   ip address 192.168.14.7/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 192.168.12.1/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   ip address 192.168.13.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 10
   router-id 192.168.12.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
```
### LEAF-1-2:
```console
!
hostname LEAF-1-2
!
interface Ethernet1
   no switchport
   ip address 192.168.14.3/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 IQqLTq4evBW2IbLOS7uR2Q==
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.14.9/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 192.168.12.2/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   ip ospf area 0.0.0.0
!
interface Loopback3
   ip address 192.168.13.2/32
!
ip routing
!
router ospf 10
   router-id 192.168.12.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
```
### LEAF-1-3:
```console
!
hostname LEAF-1-3
!
interface Ethernet1
   no switchport
   ip address 192.168.14.5/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 IQqLTq4evBW2IbLOS7uR2Q==
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.14.11/31
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 qHqUlYx8jnPAy5SWZRj0Vw==
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 192.168.12.3/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   ip address 192.168.13.3/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 10
   router-id 192.168.12.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
```

## Вывод комманд маршрутизации
### SPINE-1-1:
```console
sh bfd peers
VRF name: default
-----------------
DstAddr          MyDisc   YourDisc Interface/Transport    Type          LastUp 
------------ ---------- ---------- -------------------- ------- ---------------
192.168.14.1 1699624219 3974737377       Ethernet1(14)  normal  11/26/24 18:55 
192.168.14.3 1900157395 4088308666       Ethernet2(15)  normal  11/26/24 18:51 
192.168.14.5 1331653989 2920999305       Ethernet3(16)  normal  11/26/24 18:51 
!
sh ip route ospf
VRF: default
 O        192.168.10.2/32 [110/30] via 192.168.14.1, Ethernet1
                                   via 192.168.14.3, Ethernet2
                                   via 192.168.14.5, Ethernet3
 O        192.168.12.1/32 [110/20] via 192.168.14.1, Ethernet1
 O        192.168.12.2/32 [110/20] via 192.168.14.3, Ethernet2
 O        192.168.12.3/32 [110/20] via 192.168.14.5, Ethernet3
 O        192.168.13.1/32 [110/20] via 192.168.14.1, Ethernet1
 O        192.168.13.3/32 [110/20] via 192.168.14.5, Ethernet3
 O        192.168.14.6/31 [110/20] via 192.168.14.1, Ethernet1
 O        192.168.14.8/31 [110/20] via 192.168.14.3, Ethernet2
 O        192.168.14.10/31 [110/20] via 192.168.14.5, Ethernet3
 !
 ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2) 72(100) bytes of data.
80 bytes from 192.168.10.2: icmp_seq=1 ttl=63 time=23.4 ms
80 bytes from 192.168.10.2: icmp_seq=2 ttl=63 time=28.4 ms
80 bytes from 192.168.10.2: icmp_seq=3 ttl=63 time=35.9 ms
80 bytes from 192.168.10.2: icmp_seq=4 ttl=63 time=21.8 ms
80 bytes from 192.168.10.2: icmp_seq=5 ttl=63 time=24.5 ms

--- 192.168.10.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 79ms
rtt min/avg/max/mdev = 21.846/26.844/35.900/5.024 ms, pipe 3, ipg/ewma 19.939/25
```
### SPINE-1-2:
```console
sh bfd peers
VRF name: default
-----------------
DstAddr           MyDisc   YourDisc Interface/Transport   Type          LastUp 
------------- ---------- ---------- ------------------- ------- ---------------
192.168.14.7   818084205 1407031011       Ethernet1(14) normal  11/26/24 18:39 
192.168.14.9  3622128429 1833402391       Ethernet2(15) normal  11/26/24 18:30 
192.168.14.11  133838506 2722411222       Ethernet3(16) normal  11/26/24 18:30 
!
sh ip ospf neig
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
192.168.12.1    10       default  0   FULL                   00:00:38    192.168.14.7    Ethernet1
192.168.12.2    10       default  1   FULL                   00:00:36    192.168.14.9    Ethernet2
192.168.12.3    10       default  1   FULL                   00:00:34    192.168.14.11   Ethernet3
!
SPINE-1-2#sh ip route ospf
sh ip route ospf
VRF: default
 O        192.168.10.1/32 [110/30] via 192.168.14.7, Ethernet1
                                   via 192.168.14.9, Ethernet2
                                   via 192.168.14.11, Ethernet3
 O        192.168.12.1/32 [110/20] via 192.168.14.7, Ethernet1
 O        192.168.12.2/32 [110/20] via 192.168.14.9, Ethernet2
 O        192.168.12.3/32 [110/20] via 192.168.14.11, Ethernet3
 O        192.168.13.1/32 [110/20] via 192.168.14.7, Ethernet1
 O        192.168.13.3/32 [110/20] via 192.168.14.11, Ethernet3
 O        192.168.14.0/31 [110/20] via 192.168.14.7, Ethernet1
 O        192.168.14.2/31 [110/20] via 192.168.14.9, Ethernet2
 O        192.168.14.4/31 [110/20] via 192.168.14.11, Ethernet3
 !
 ping 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 72(100) bytes of data.
80 bytes from 192.168.10.1: icmp_seq=1 ttl=63 time=18.2 ms
80 bytes from 192.168.10.1: icmp_seq=2 ttl=63 time=26.1 ms
80 bytes from 192.168.10.1: icmp_seq=3 ttl=63 time=27.1 ms
80 bytes from 192.168.10.1: icmp_seq=4 ttl=63 time=20.0 ms
80 bytes from 192.168.10.1: icmp_seq=5 ttl=63 time=14.3 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 83ms
rtt min/avg/max/mdev = 14.388/21.203/27.149/4.816 ms, pipe 2, ipg/ewma 20.944/19.495 ms
```
### LEAF-1-1:
```console
sh bfd peers
VRF name: default
-----------------
DstAddr          MyDisc   YourDisc Interface/Transport    Type          LastUp 
------------ ---------- ---------- -------------------- ------- ---------------
192.168.14.0  751627048 2419526745       Ethernet1(15)  normal  11/26/24 19:00 
192.168.14.6 4182183414 1128295576       Ethernet2(16)  normal  11/26/24 18:51 
!
sh ip ospf neig
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address
192.168.10.2    10       default  0   FULL                   00:00:36    192.168
192.168.10.1    10       default  0   FULL                   00:00:29    192.168
```
### LEAF-1-2:
```console
sh bfd peers
VRF name: default
-----------------
DstAddr          MyDisc   YourDisc Interface/Transport    Type          LastUp 
------------ ---------- ---------- -------------------- ------- ---------------
192.168.14.2 1263130245 3547025372       Ethernet1(15)  normal  11/26/24 19:00 
192.168.14.8   29960964  437707522       Ethernet2(16)  normal  11/26/24 19:00 
!
sh ip ospf neig
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
192.168.10.1    10       default  0   FULL                   00:00:31    192.168.14.2    Ethernet1
192.168.10.2    10       default  0   FULL                   00:00:32    192.168.14.8    Ethernet2
```
### LEAF-1-3:
```console
sh bfd peers
VRF name: default
-----------------
DstAddr           MyDisc   YourDisc Interface/Transport   Type          LastUp 
------------- ---------- ---------- ------------------- ------- ---------------
192.168.14.4  2239808815 1835494464       Ethernet1(15) normal  11/26/24 19:00 
192.168.14.10  556523317 1387204567       Ethernet2(16) normal  11/26/24 19:00 
!
sh ip ospf nei
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
192.168.10.1    10       default  0   FULL                   00:00:37    192.168.14.4    Ethernet1
192.168.10.2    10       default  0   FULL                   00:00:32    192.168.14.10   Ethernet2
```