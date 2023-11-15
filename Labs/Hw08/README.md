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
  exit-af-topology
  network 10.112.2.16 0.0.0.3
  network 10.112.2.20 0.0.0.3
  network 10.112.3.18 0.0.0.0
  eigrp router-id 10.112.3.18
 exit-address-family
!
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
   redistribute static
  exit-af-topology
  network 10.112.2.4 0.0.0.3
  network 10.112.2.8 0.0.0.3
  network 10.112.2.20 0.0.0.3
  network 10.112.2.24 0.0.0.3
  network 10.112.3.16 0.0.0.0
  eigrp router-id 10.112.3.16
 exit-address-family
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/3
!
ip prefix-list R32 seq 5 permit 0.0.0.0/0
ip prefix-list R32 seq 10 deny 0.0.0.0/0 le 32
!
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