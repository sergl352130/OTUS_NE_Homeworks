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
10.111.3.12     10.111.3.12     1374        0x80000004 0x008B7C 5
10.111.3.13     10.111.3.13     1335        0x80000005 0x00E003 5
10.111.3.14     10.111.3.14     449         0x80000008 0x00A06C 7
10.111.3.15     10.111.3.15     1333        0x80000007 0x005EAD 7

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.111.2.0      10.111.3.12     1263        0x80000003 0x00D25D
10.111.2.4      10.111.3.13     1832        0x80000002 0x00A685
10.111.2.28     10.111.3.14     445         0x80000001 0x00B162
10.111.3.4      10.111.3.12     1263        0x80000003 0x00BB6B
10.111.3.5      10.111.3.13     1832        0x80000002 0x00AD78

                Router Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.111.3.14     10.111.3.14     165         0x80000002 0x0041FD 2
10.111.3.19     10.111.3.19     165         0x80000002 0x00C970 2

                Summary Net Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.111.3.14     317         0x80000001 0x007241
```

```
R19#show ip ospf database 

            OSPF Router with ID (10.111.3.19) (Process ID 111)

                Router Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.111.3.14     10.111.3.14     120         0x80000003 0x003FFE 2
10.111.3.19     10.111.3.19     214         0x80000003 0x00C771 2

                Summary Net Link States (Area 101)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.111.3.14     364         0x80000002 0x007042
```

+ #### Шаг 4: Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101
