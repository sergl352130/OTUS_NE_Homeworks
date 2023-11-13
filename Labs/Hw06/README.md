# Лабораторная работа 06
+ ## OSPF. Фильтрация
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw06/Network_topology_OSPF.png?raw=true)

## Цели:
+ ### Часть 1: Настройка OSPF в офисе Москва
+ ### Часть 2: Настройка разделения сети на зоны
+ ### Часть 3: Настройка фильтрации между зонами

### Этапы:
+ #### Шаг 1: Настройка OSPF зоны 0 (backbone) для маршрутизаторов R14-R15, R12-R13

#### R14:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
interface Loopback0
 description R14 Mgmt
 ip address 10.111.3.14 255.255.255.255
 ip ospf 111 area 0
!
interface Ethernet0/0
 description p2p to R12
 ip address 10.111.2.10 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/1
 description p2p to R13
 ip address 10.111.2.22 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/2
 description p2p to R15
 ip address 10.111.2.25 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
router ospf 111
 router-id 10.111.3.14
!         
```

#### R15:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R15
!
interface Loopback0
 description R15 Mgmt
 ip address 10.111.3.15 255.255.255.255
 ip ospf 111 area 0
!
interface Ethernet0/0
 description p2p to R13
 ip address 10.111.2.18 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/1
 description p2p to R12
 ip address 10.111.2.14 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/2
 description p2p to R14
 ip address 10.111.2.26 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
router ospf 111
 router-id 10.111.3.15
!         
```

#### R12:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R12
!
interface Loopback0
 description R12 Mgmt
 ip address 10.111.3.12 255.255.255.255
 ip ospf 111 area 0
!
interface Ethernet0/0
 description p2p to R14
 ip address 10.111.2.9 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/1
 description p2p to R15
 ip address 10.111.2.13 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
router ospf 111
 router-id 10.111.3.12
!         
```

#### R13:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R13
!
interface Loopback0
 description R13 Mgmt
 ip address 10.111.3.13 255.255.255.255
 ip ospf 111 area 0
!
interface Ethernet0/0
 description p2p to R15
 ip address 10.111.2.17 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/1
 description p2p to R14
 ip address 10.111.2.21 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
router ospf 111
 router-id 10.111.3.13
!         
```

+ #### Шаг 2: Настройка OSPF зоны 10 для маршрутизаторов  R12-R13. Дополнительно к маршрутам должны получать маршрут по умолчанию

#### R12:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R12
!
interface Ethernet1/0
 description p2p to SW4
 ip address 10.111.2.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 10
!
router ospf 111
 router-id 10.111.3.12
!         
```

#### R13:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R13
!
interface Ethernet1/0
 description p2p to SW5
 ip address 10.111.2.6 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 10
!
router ospf 111
 router-id 10.111.3.13
!         
```

#### SW4:

```
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW4
!
interface Loopback0
 description SW4 Mgmt
 ip address 10.111.3.4 255.255.255.255
 ip ospf 111 area 10
!
interface Ethernet1/0
 description p2p to R12
 no switchport
 ip address 10.111.2.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 10
 duplex auto
!
router ospf 111
 router-id 10.111.3.4
!         
```

#### SW5:

```
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW5
!
interface Loopback0
 description SW5 Mgmt
 ip address 10.111.3.5 255.255.255.255
 ip ospf 111 area 10
!
interface Ethernet1/0
 description p2p to R13
 no switchport
 ip address 10.111.2.5 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 10
 duplex auto
!
router ospf 111
 router-id 10.111.3.5
!         
```

+ #### Шаг 3: Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию

#### R14:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
interface Ethernet0/3
 description p2p to R19
 ip address 10.111.2.29 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 101
!
router ospf 111
 router-id 10.111.3.14
 area 101 stub no-summary
!         
```

#### R19:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R19
!
interface Loopback0
 description R19 Mgmt
 ip address 10.111.3.19 255.255.255.255
 ip ospf 111 area 101
!
interface Ethernet0/3
 description p2p to R14
 ip address 10.111.2.30 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 101
!
router ospf 111
 router-id 10.111.3.19
 area 101 stub
!         
```

```
R14#show ip ospf database

            OSPF Router with ID (10.111.3.14) (Process ID 111)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.111.3.12     10.111.3.12     892         0x80000007 0x00857F 5
10.111.3.13     10.111.3.13     904         0x80000007 0x00DC05 5
10.111.3.14     10.111.3.14     884         0x80000007 0x009677 7
10.111.3.15     10.111.3.15     919         0x8000000C 0x0024E1 7

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.111.2.0      10.111.3.12     892         0x80000005 0x00CE5F
10.111.2.4      10.111.3.13     904         0x80000005 0x00A088
10.111.2.28     10.111.3.14     884         0x80000005 0x00A966
10.111.2.32     10.111.3.15     919         0x80000003 0x007F8D
10.111.3.4      10.111.3.12     892         0x80000005 0x00B76D
10.111.3.5      10.111.3.13     904         0x80000005 0x00A77B
10.111.3.19     10.111.3.14     331         0x80000001 0x001DFA

                Router Link States (Area 101)
          
Link ID         ADV Router      Age         Seq#       Checksum Link count
10.111.3.14     10.111.3.14     884         0x80000006 0x003902 2
10.111.3.19     10.111.3.19     337         0x80000007 0x008212 3

                Summary Net Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.111.3.14     884         0x80000005 0x006A45
R14#show ip ospf database summary 0.0.0.0

            OSPF Router with ID (10.111.3.14) (Process ID 111)

                Summary Net Link States (Area 101)

  LS age: 906
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 0.0.0.0 (summary Network Number)
  Advertising Router: 10.111.3.14
  LS Seq Number: 80000005
  Checksum: 0x6A45
  Length: 28
  Network Mask: /0
        MTID: 0         Metric: 1 
```

```
R19#show ip ospf database

            OSPF Router with ID (10.111.3.19) (Process ID 111)

                Router Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.111.3.14     10.111.3.14     733         0x80000006 0x003902 2
10.111.3.19     10.111.3.19     184         0x80000007 0x008212 3

                Summary Net Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.111.3.14     733         0x80000005 0x006A45
R19#show ip ospf database summary 0.0.0.0

            OSPF Router with ID (10.111.3.19) (Process ID 111)

                Summary Net Link States (Area 101)

  Routing Bit Set on this LSA in topology Base with MTID 0
  LS age: 828
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 0.0.0.0 (summary Network Number)
  Advertising Router: 10.111.3.14
  LS Seq Number: 80000005
  Checksum: 0x6A45
  Length: 28
  Network Mask: /0
        MTID: 0         Metric: 1 
```

+ #### Шаг 4: Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101

#### R15:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R15
!
interface Ethernet0/3
 description p2p to R20
 ip address 10.111.2.33 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 102
!
router ospf 111
 router-id 10.111.3.15
!         
ip prefix-list NOarea101 seq 5 deny 10.111.2.28/30
ip prefix-list NOarea101 seq 10 deny 10.111.3.19/32
ip prefix-list NOarea101 seq 15 permit 0.0.0.0/0 le 32
!
```

#### R20:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R20
!
interface Ethernet0/3
 description p2p to R15
 ip address 10.111.2.34 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 102
!
router ospf 111
 router-id 10.111.3.20
!         
```

```
R20#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 18 subnets, 2 masks
O IA     10.111.2.0/30 [110/30] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.2.4/30 [110/30] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.2.8/30 [110/30] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.2.12/30 [110/20] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.2.16/30 [110/20] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.2.20/30 [110/30] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.2.24/30 [110/20] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     **10.111.2.28/30 [110/30] via 10.111.2.33, 00:02:38, Ethernet0/3**
C        10.111.2.32/30 is directly connected, Ethernet0/3
L        10.111.2.34/32 is directly connected, Ethernet0/3
O IA     10.111.3.4/32 [110/31] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.3.5/32 [110/31] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.3.12/32 [110/21] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.3.13/32 [110/21] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.3.14/32 [110/21] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     10.111.3.15/32 [110/11] via 10.111.2.33, 01:17:20, Ethernet0/3
O IA     **10.111.3.19/32 [110/31] via 10.111.2.33, 00:00:26, Ethernet0/3**
C        10.111.3.20/32 is directly connected, Loopback0
```

```
R15#configure terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
R15(config)#router ospf 111
R15(config-router)#area 102 filter-list prefix NOarea101 in
```

```
R20#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 16 subnets, 2 masks
O IA     10.111.2.0/30 [110/30] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.2.4/30 [110/30] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.2.8/30 [110/30] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.2.12/30 [110/20] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.2.16/30 [110/20] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.2.20/30 [110/30] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.2.24/30 [110/20] via 10.111.2.33, 01:29:40, Ethernet0/3
C        10.111.2.32/30 is directly connected, Ethernet0/3
L        10.111.2.34/32 is directly connected, Ethernet0/3
O IA     10.111.3.4/32 [110/31] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.3.5/32 [110/31] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.3.12/32 [110/21] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.3.13/32 [110/21] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.3.14/32 [110/21] via 10.111.2.33, 01:29:40, Ethernet0/3
O IA     10.111.3.15/32 [110/11] via 10.111.2.33, 01:29:40, Ethernet0/3
C        10.111.3.20/32 is directly connected, Loopback0
```