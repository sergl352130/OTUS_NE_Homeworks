# Лабораторная работа 12
+ ## Основные протоколы сети Интернет
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw12/Network_topology.png?raw=true)

## Цели:
+ ### Настроить DHCP для офиса Москва
+ ### Настроить синхронизацию времени для офиса Москва
+ ### Настроить NAT для офисов Москва, C.-Петербург и Чокурдах


### Этапы:
+ #### Шаг 1: Настроить NAT (PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001
+ #### Шаг 2: Настроить NAT (PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042
+ #### Шаг 3: Настроить статический NAT для R20
+ #### Шаг 4: Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления
+ #### Шаг 5: Настроить статический NAT (PAT) для офиса Чокурдах
+ #### Шаг 6: Настроить для IPv4 DHCP-сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP
+ #### Шаг 7: Настроить NTP-сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13
+ #### Шаг 8: Все офисы в лабораторной работе должны иметь IP связность 


#### R14:

```
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
interface Ethernet0/2
 description p2p to R15
 ip address 10.111.2.25 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet1/0
 description to R22 Kitorn
 ip address 22.111.22.14 255.255.255.0
!
router bgp 1001
 bgp router-id 10.111.3.14
 bgp log-neighbor-changes
 network 10.111.3.14 mask 255.255.255.255
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
!
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
!
route-map R22-Kitorn-OUT permit 10
 match as-path 1
 set as-path prepend 1001 1001
!
```

```       

```

#### R15:

```
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
interface Ethernet0/2
 description p2p to R14
 ip address 10.111.2.26 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet1/0
 description to R21 Lamas
 ip address 33.111.21.15 255.255.255.0
!
router bgp 1001
 bgp router-id 10.111.3.15
 bgp log-neighbor-changes
 network 10.111.3.15 mask 255.255.255.255
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
 neighbor 33.111.21.21 route-map R21-Lamas-IN in
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
!
ip prefix-list NOarea101 seq 5 deny 10.111.2.28/30
ip prefix-list NOarea101 seq 10 deny 10.111.3.19/32
ip prefix-list NOarea101 seq 15 permit 0.0.0.0/0 le 32
!
route-map R21-Lamas-IN permit 10
 set local-preference 1000
!
route-map R21-Lamas-OUT permit 10
 match as-path 1
!
```

```       

```

#### R18:

```
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
interface Ethernet1/0
 description to R24 Triada
 ip address 44.112.24.18 255.255.255.0
!
interface Ethernet1/1
 description to R26 Triada
 ip address 44.112.26.18 255.255.255.0
!
router bgp 2042
 bgp router-id 10.112.3.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 10.112.3.18 mask 255.255.255.255
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
route-map Triada-OUT permit 10
 match ip address prefix-list Triada-OUT
!
```

```

```

#### R21:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R21
!
interface Loopback0
 description R21 Mgmt
 ip address 10.33.33.21 255.255.255.255
!
interface Ethernet0/0
 description to R24 Triada
 ip address 44.33.24.21 255.255.255.0
!
interface Ethernet0/1
 description to R22 Kitorn
 ip address 33.22.21.21 255.255.255.0
!
interface Ethernet1/0
 description to R15 Moscow
 ip address 33.111.21.21 255.255.255.0
!
router bgp 301
 bgp router-id 10.33.33.21
 bgp log-neighbor-changes
 network 10.33.33.21 mask 255.255.255.255
 neighbor 33.22.21.22 remote-as 101
 neighbor 33.111.21.15 remote-as 1001
 neighbor 33.111.21.15 default-originate
 neighbor 33.111.21.15 route-map Lamas-OUT out
 neighbor 44.33.24.24 remote-as 520
!
ip as-path access-list 1 permit .*_2042$
ip as-path access-list 1 deny .*
!
route-map Lamas-OUT permit 10
 match as-path 1
!
```

```

```

#### R22:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R22
!
interface Loopback0
 description R22 Mgmt
 ip address 10.22.22.22 255.255.255.255
!         
interface Ethernet0/0
 description to R23 Triada
 ip address 44.22.23.22 255.255.255.0
!
interface Ethernet0/1
 description to R21 Lamas
 ip address 33.22.21.22 255.255.255.0
!
interface Ethernet1/0
 description to R14 Moscow
 ip address 22.111.22.22 255.255.255.0
!
router bgp 101
 bgp router-id 10.22.22.22
 bgp log-neighbor-changes
 network 10.22.22.22 mask 255.255.255.255
 neighbor 22.111.22.14 remote-as 1001
 neighbor 22.111.22.14 default-originate
 neighbor 22.111.22.14 route-map Nothing-OUT out
 neighbor 33.22.21.21 remote-as 301
 neighbor 44.22.23.23 remote-as 520
!
ip prefix-list Nothing-OUT seq 5 deny 0.0.0.0/0 le 32
!
route-map Nothing-OUT permit 10
 match ip address prefix-list Nothing-OUT
!
```

```

```