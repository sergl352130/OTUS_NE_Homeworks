# Лабораторная работа 08
+ ## EIGRP
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw08/Network_topology_EIGRP.png?raw=true)

## Цели:
+ ### Настройка EIGRP в офисе Санкт-Петербург

### Этапы:
+ #### Шаг 1: В офисе С.-Петербург настроить EIGRP. Для настройки сети использовать EIGRP named-mode
+ #### Шаг 2: R16 и R17 анонсируют только суммарные префиксы
+ #### Шаг 3: R32 получает только маршрут по умолчанию

#### R18:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R18
!
interface Loopback0
 description R18 Mgmt
 ip address 10.112.3.18 255.255.255.255
!
interface Ethernet0/0
 description p2p to R17
 ip address 10.112.2.18 255.255.255.252
!
interface Ethernet0/1
 description p2p to R16
 ip address 10.112.2.22 255.255.255.252
!
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 112
  !
  topology base
   redistribute static
  exit-af-topology
  network 10.112.2.16 0.0.0.3
  network 10.112.2.20 0.0.0.3
  network 10.112.3.18 0.0.0.0
  eigrp router-id 10.112.3.18
 exit-address-family
!
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26 5
!
```

```
R18#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 44.112.24.24 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 44.112.24.24
      10.0.0.0/8 is variably subnetted, 12 subnets, 3 masks
D        10.112.0.0/24 [90/1541120] via 10.112.2.17, 00:48:59, Ethernet0/0
D        10.112.2.0/24 [90/1536000] via 10.112.2.21, 00:49:29, Ethernet0/1
                       [90/1536000] via 10.112.2.17, 00:49:29, Ethernet0/0
C        10.112.2.16/30 is directly connected, Ethernet0/0
L        10.112.2.18/32 is directly connected, Ethernet0/0
C        10.112.2.20/30 is directly connected, Ethernet0/1
L        10.112.2.22/32 is directly connected, Ethernet0/1
D        10.112.3.9/32 [90/1536640] via 10.112.2.17, 00:49:24, Ethernet0/0
D        10.112.3.10/32 [90/1536640] via 10.112.2.21, 00:49:24, Ethernet0/1
D        10.112.3.16/32 [90/1024640] via 10.112.2.21, 00:49:24, Ethernet0/1
D        10.112.3.17/32 [90/1024640] via 10.112.2.17, 00:49:24, Ethernet0/0
C        10.112.3.18/32 is directly connected, Loopback0
D        10.112.3.32/32 [90/1536640] via 10.112.2.21, 00:49:24, Ethernet0/1
      44.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        44.112.24.0/24 is directly connected, Ethernet1/0
L        44.112.24.18/32 is directly connected, Ethernet1/0
C        44.112.26.0/24 is directly connected, Ethernet1/1
L        44.112.26.18/32 is directly connected, Ethernet1/1

R18#
R18#show ip protocols 
*** IP Routing is NSF aware ***

Routing Protocol is "eigrp 112"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  Redistributing: static
  EIGRP-IPv4 VR(SPB) Address-Family Protocol for AS(112)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    NSF-aware route hold timer is 240
    Router-ID: 10.112.3.18
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 11
      Total Redist Count: 1

  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    10.112.2.16/30
    10.112.2.20/30
    10.112.3.18/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.112.2.17           90      00:20:17
    10.112.2.21           90      00:20:17
  Distance: internal 90 external 170
```

#### R17:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R17
!
interface Loopback0
 description R17 Mgmt
 ip address 10.112.3.17 255.255.255.255
!
interface Ethernet0/0
 description p2p to R18
 ip address 10.112.2.17 255.255.255.252
!
interface Ethernet0/2
 description p2p to R16
 ip address 10.112.2.5 255.255.255.252
!
interface Ethernet1/0
 description p2p to SW9
 ip address 10.112.2.2 255.255.255.252
!
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 112
  !
  af-interface Ethernet0/0
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !
  af-interface Ethernet0/2
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !
  af-interface Ethernet1/0
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.112.2.0 0.0.0.3
  network 10.112.2.4 0.0.0.3
  network 10.112.2.16 0.0.0.3
  network 10.112.3.17 0.0.0.0
  eigrp router-id 10.112.3.17
 exit-address-family
!
```

```
R17#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.112.2.18 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/1536000] via 10.112.2.18, 00:05:42, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 15 subnets, 3 masks
D        10.112.0.0/24 [90/1029120] via 10.112.2.1, 00:51:58, Ethernet1/0
D        10.112.2.0/24 is a summary, 00:52:29, Null0
C        10.112.2.0/30 is directly connected, Ethernet1/0
L        10.112.2.2/32 is directly connected, Ethernet1/0
C        10.112.2.4/30 is directly connected, Ethernet0/2
L        10.112.2.5/32 is directly connected, Ethernet0/2
C        10.112.2.16/30 is directly connected, Ethernet0/0
L        10.112.2.17/32 is directly connected, Ethernet0/0
D        10.112.2.20/30 [90/1536000] via 10.112.2.18, 00:52:29, Ethernet0/0
D        10.112.3.9/32 [90/1024640] via 10.112.2.1, 00:52:25, Ethernet1/0
D        10.112.3.10/32 [90/1536640] via 10.112.2.6, 00:52:23, Ethernet0/2
D        10.112.3.16/32 [90/1024640] via 10.112.2.6, 00:52:23, Ethernet0/2
C        10.112.3.17/32 is directly connected, Loopback0
D        10.112.3.18/32 [90/1024640] via 10.112.2.18, 00:52:25, Ethernet0/0
D        10.112.3.32/32 [90/1536640] via 10.112.2.6, 00:52:23, Ethernet0/2

R17#
R17#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "eigrp 112"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(SPB) Address-Family Protocol for AS(112)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    NSF-aware route hold timer is 240
    Router-ID: 10.112.3.17
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 13
      Total Redist Count: 0

  Automatic Summarization: disabled
  Address Summarization:
    10.112.2.0/24 for Et1/0, Et0/2, Et0/0
      Summarizing 4 components with metric 131072000
  Maximum path: 4
  Routing for Networks:
    10.112.2.0/30
    10.112.2.4/30
    10.112.2.16/30
    10.112.3.17/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.112.2.18           90      00:05:58
    10.112.2.1            90      00:05:58
    10.112.2.6            90      00:05:58
  Distance: internal 90 external 170
```

#### R16:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R16
!
interface Loopback0
 description R16 Mgmt
 ip address 10.112.3.16 255.255.255.255
!
interface Ethernet0/1
 description p2p to R18
 ip address 10.112.2.21 255.255.255.252
!
interface Ethernet0/2
 description p2p to R17
 ip address 10.112.2.6 255.255.255.252
!
interface Ethernet0/3
 description p2p to R32
 ip address 10.112.2.25 255.255.255.252
!
interface Ethernet1/0
 description p2p to SW10
 ip address 10.112.2.10 255.255.255.252
!
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 112
  !
  af-interface Ethernet0/1
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !
  af-interface Ethernet0/2
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !       
  af-interface Ethernet0/3
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !
  af-interface Ethernet1/0
   summary-address 10.112.2.0 255.255.255.0
  exit-af-interface
  !
  topology base
   default-metric 10000 100 255 1 1500
   distribute-list prefix R32 out Ethernet0/3   
  exit-af-topology
  network 10.112.2.4 0.0.0.3
  network 10.112.2.8 0.0.0.3
  network 10.112.2.20 0.0.0.3
  network 10.112.2.24 0.0.0.3
  network 10.112.3.16 0.0.0.0
  eigrp router-id 10.112.3.16
 exit-address-family
!
ip prefix-list R32 seq 5 permit 0.0.0.0/0
ip prefix-list R32 seq 10 deny 0.0.0.0/0 le 32
!
```

```
R16#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.112.2.22 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/1536000] via 10.112.2.22, 00:09:02, Ethernet0/1
      10.0.0.0/8 is variably subnetted, 17 subnets, 3 masks
D        10.112.0.0/24 [90/1541120] via 10.112.2.5, 00:55:19, Ethernet0/2
D        10.112.2.0/24 is a summary, 00:55:49, Null0
C        10.112.2.4/30 is directly connected, Ethernet0/2
L        10.112.2.6/32 is directly connected, Ethernet0/2
C        10.112.2.8/30 is directly connected, Ethernet1/0
L        10.112.2.10/32 is directly connected, Ethernet1/0
D        10.112.2.16/30 [90/1536000] via 10.112.2.22, 00:55:49, Ethernet0/1
C        10.112.2.20/30 is directly connected, Ethernet0/1
L        10.112.2.21/32 is directly connected, Ethernet0/1
C        10.112.2.24/30 is directly connected, Ethernet0/3
L        10.112.2.25/32 is directly connected, Ethernet0/3
D        10.112.3.9/32 [90/1536640] via 10.112.2.5, 00:55:45, Ethernet0/2
D        10.112.3.10/32 [90/1024640] via 10.112.2.9, 00:55:44, Ethernet1/0
C        10.112.3.16/32 is directly connected, Loopback0
D        10.112.3.17/32 [90/1024640] via 10.112.2.5, 00:55:49, Ethernet0/2
D        10.112.3.18/32 [90/1024640] via 10.112.2.22, 00:55:46, Ethernet0/1
D        10.112.3.32/32 [90/1024640] via 10.112.2.26, 00:55:44, Ethernet0/3
R16#
R16#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "eigrp 112"
  Outgoing update filter list for all interfaces is not set
    Ethernet0/3 filtered by (prefix-list) R32 (per-user), default is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(SPB) Address-Family Protocol for AS(112)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    NSF-aware route hold timer is 240
    Router-ID: 10.112.3.16
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Default redistribution metric is 10000 100 255 1 1500
      Total Prefix Count: 14
      Total Redist Count: 0

  Automatic Summarization: disabled
  Address Summarization:
    10.112.2.0/24 for Et0/2, Et1/0, Et0/1
        Et0/3
      Summarizing 5 components with metric 131072000
  Maximum path: 4
  Routing for Networks:
    10.112.2.4/30
    10.112.2.8/30
    10.112.2.20/30
    10.112.2.24/30
    10.112.3.16/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.112.2.26           90      00:09:11
    10.112.2.22           90      00:09:11
    10.112.2.9            90      00:09:11
    10.112.2.5            90      00:09:11
  Distance: internal 90 external 170
```

#### R32:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R32
!
interface Loopback0
 description R32 Mgmt
 ip address 10.112.3.32 255.255.255.255
!
interface Ethernet0/3
 description p2p to R16
 ip address 10.112.2.26 255.255.255.252
!
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 112
  !
  topology base
  exit-af-topology
  network 10.112.2.24 0.0.0.3
  network 10.112.3.32 0.0.0.0
  eigrp router-id 10.112.3.32
 exit-address-family
!
```

```
R32#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.112.2.25 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/2048000] via 10.112.2.25, 00:13:27, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.112.2.24/30 is directly connected, Ethernet0/3
L        10.112.2.26/32 is directly connected, Ethernet0/3
C        10.112.3.32/32 is directly connected, Loopback0
R32#
R32#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "eigrp 112"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(SPB) Address-Family Protocol for AS(112)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    NSF-aware route hold timer is 240
    Router-ID: 10.112.3.32
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 3
      Total Redist Count: 0

  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    10.112.2.24/30
    10.112.3.32/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.112.2.25           90      00:13:38
  Distance: internal 90 external 170
```

#### SW9:

```
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW9
!
interface Loopback0
 description SW9 Mgmt
 ip address 10.112.3.9 255.255.255.255
!
interface Ethernet1/0
 description p2p to R17
 no switchport
 ip address 10.112.2.1 255.255.255.252
 duplex auto
!
interface Vlan18
 description Default gateway VLAN18
 ip address 10.112.0.1 255.255.255.0
!
router eigrp SPB
 !        
 address-family ipv4 unicast autonomous-system 112
  !
  topology base
  exit-af-topology
  network 10.112.0.0 0.0.0.255
  network 10.112.2.0 0.0.0.3
  network 10.112.3.9 0.0.0.0
  eigrp router-id 10.112.3.9
 exit-address-family
!
```

```
SW9#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.112.2.2 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/2048000] via 10.112.2.2, 00:17:57, Ethernet1/0
      10.0.0.0/8 is variably subnetted, 11 subnets, 3 masks
C        10.112.0.0/24 is directly connected, Vlan18
L        10.112.0.1/32 is directly connected, Vlan18
D        10.112.2.0/24 [90/1536000] via 10.112.2.2, 01:04:40, Ethernet1/0
C        10.112.2.0/30 is directly connected, Ethernet1/0
L        10.112.2.1/32 is directly connected, Ethernet1/0
C        10.112.3.9/32 is directly connected, Loopback0
D        10.112.3.10/32 [90/2048640] via 10.112.2.2, 01:04:40, Ethernet1/0
D        10.112.3.16/32 [90/1536640] via 10.112.2.2, 01:04:40, Ethernet1/0
D        10.112.3.17/32 [90/1024640] via 10.112.2.2, 01:04:40, Ethernet1/0
D        10.112.3.18/32 [90/1536640] via 10.112.2.2, 01:04:40, Ethernet1/0
          
SW9#                               
SW9#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "eigrp 112"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(SPB) Address-Family Protocol for AS(112)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 10.112.3.9
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 10
      Total Redist Count: 0

  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    10.112.0.0/24
    10.112.2.0/30
    10.112.3.9/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.112.2.2            90      00:18:30
  Distance: internal 90 external 170
```

#### SW10:

```
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW10
!
interface Loopback0
 description SW10 Mgmt
 ip address 10.112.3.10 255.255.255.255
!
interface Ethernet1/0
 description p2p to R16
 no switchport
 ip address 10.112.2.9 255.255.255.252
 duplex auto
!
interface Vlan10
 description Default gateway VLAN10
 ip address 10.112.1.1 255.255.255.0
!
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 112
  !
  topology base
  exit-af-topology
  network 10.112.1.0 0.0.0.255
  network 10.112.2.8 0.0.0.3
  network 10.112.3.10 0.0.0.0
  eigrp router-id 10.112.3.10
 exit-address-family
!
```

```
SW10#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.112.2.10 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/2048000] via 10.112.2.10, 00:19:24, Ethernet1/0
      10.0.0.0/8 is variably subnetted, 12 subnets, 3 masks
D        10.112.0.0/24 [90/2053120] via 10.112.2.10, 01:05:40, Ethernet1/0
C        10.112.1.0/24 is directly connected, Vlan10
L        10.112.1.1/32 is directly connected, Vlan10
D        10.112.2.0/24 [90/1536000] via 10.112.2.10, 01:06:10, Ethernet1/0
C        10.112.2.8/30 is directly connected, Ethernet1/0
L        10.112.2.9/32 is directly connected, Ethernet1/0
D        10.112.3.9/32 [90/2048640] via 10.112.2.10, 01:06:05, Ethernet1/0
C        10.112.3.10/32 is directly connected, Loopback0
D        10.112.3.16/32 [90/1024640] via 10.112.2.10, 01:06:05, Ethernet1/0
D        10.112.3.17/32 [90/1536640] via 10.112.2.10, 01:06:05, Ethernet1/0
D        10.112.3.18/32 [90/1536640] via 10.112.2.10, 01:06:05, Ethernet1/0
D        10.112.3.32/32 [90/1536640] via 10.112.2.10, 01:06:05, Ethernet1/0
SW10#
SW10#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "eigrp 112"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(SPB) Address-Family Protocol for AS(112)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 10.112.3.10
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 10
      Total Redist Count: 0

  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    10.112.2.8/30
    10.112.3.10/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.112.2.10           90      00:19:34
  Distance: internal 90 external 170
```