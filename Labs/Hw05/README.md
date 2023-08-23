# Лабораторная работа 05
+ ## Маршрутизация на основе политик (PBR)
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw04/Network_topology.png?raw=true)

## Цели:
+ ### Часть 1: Настройка правил образования политик маршрутизации
+ ### Часть 2: Настройка выбора маршрута на основе политики
+ ### Часть 3: Настройка отслеживания падений каналов

### IP-адресация офисов Чокурдах и Лабытнанги и провайдера Триада в части сопряжения с указанными офисами

|Device|Interface|IP Address   |Subnet Mask    |Description   |
|:----:|:--------|:------------|:--------------|:------------:|
|Чокурдах|
|VPC30 |eth0     |10.111.4.66  |255.255.255.192|VPC30         |
|VPC31 |eth0     |10.111.4.131 |255.255.255.192|VPC31         |
|SW29  |VLAN111	 |10.111.4.29  |255.255.255.248|SW29 Mgmt     |
|R28   |Loopback0|10.111.4.228 |255.255.255.255|R28 Mgmt      |
|R28   |E0/0.30  |10.111.4.65  |255.255.255.192|LAN VPC30     |
|R28   |E0/0.31  |10.111.4.129 |255.255.255.192|LAN VPC31     |
|R28   |E0/0.111 |10.111.4.28  |255.255.255.248|SW29 Mgmt     |
|R28   |E1/0     |44.114.26.28 |255.255.255.0  |to R26 Triada |
|R28   |E1/1     |44.114.25.28 |255.255.255.0  |to R25 Triada |
|Лабытнанги|
|R27   |Loopback0|10.111.5.27  |255.255.255.255|R27 Mgmt      |
|R27   |E0/0     |10.111.5.129 |255.255.255.128|internal LAN  |
|R27   |E1/0     |44.115.25.27 |255.255.255.0  |to R25 Triada |
|Триада|
|R25   |Loopback0|10.44.44.25  |255.255.255.255|R25 Mgmt      |
|R25   |E0/1     |10.44.44.2   |255.255.255.252|p2p to R23    |
|R25   |E0/2     |10.44.44.13  |255.255.255.252|p2p to R26    |
|R25   |E1/0     |44.115.25.25 |255.255.255.0  |to R27 Lab-gi |
|R25   |E1/1     |44.114.25.25 |255.255.255.0  |to R28 Chok   |
|R26   |Loopback0|10.44.44.26  |255.255.255.255|R26 Mgmt      |
|R26   |E0/1     |10.44.44.10  |255.255.255.252|p2p to R24    |
|R26   |E0/2     |10.44.44.14  |255.255.255.252|p2p to R25    |
|R26   |E1/0     |44.114.26.26 |255.255.255.0  |to R28 Chok   |
|R26   |E1/1     |44.112.26.26 |255.255.255.0  |to R18 SPb    |

### Этапы:
+ #### Шаг 1: Настройка политики маршрутизации для сетей офиса Чокурдах
+ #### Шаг 2: Настройка распределения трафика между двумя линками с провайдером
+ #### Шаг 3: Настройка отслеживания линка через технологию IP SLA (только для IPv4)

#### Принятые правила распределения трафика:
1. 


#### R28:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R28
!
!
track 10 ip sla 10 reachability
 delay down 90 up 90
!
track 20 ip sla 20 reachability
 delay down 90 up 90
!
!
interface Loopback0
 ip address 10.111.4.228 255.255.255.255
!
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.30
 description LAN VPC30
 encapsulation dot1Q 30
 ip address 10.111.4.65 255.255.255.192
 ip policy route-map VLAN30
!
interface Ethernet0/0.31
 description LAN VPC31
 encapsulation dot1Q 31
 ip address 10.111.4.129 255.255.255.192
 ip policy route-map VLAN31
!
interface Ethernet0/0.77
 encapsulation dot1Q 77 native
!
interface Ethernet0/0.111
 description SW29 Mgmt
 encapsulation dot1Q 111
 ip address 10.111.4.28 255.255.255.248
 ip policy route-map VLAN111
!
!
interface Ethernet1/0
 description to R26 Triada
 ip address 44.114.26.28 255.255.255.0
!
interface Ethernet1/1
 description to R25 Triada
 ip address 44.114.25.28 255.255.255.0
!
!
ip route 0.0.0.0 0.0.0.0 44.114.26.26 40 track 10
ip route 0.0.0.0 0.0.0.0 44.114.25.25 50 track 20
!
ip access-list standard VLAN111
 permit 10.111.4.24 0.0.0.7
ip access-list standard VLAN30
 permit 10.111.4.64 0.0.0.63
ip access-list standard VLAN31
 permit 10.111.4.128 0.0.0.63
!
ip sla 10
 icmp-echo 44.114.26.26 source-ip 44.114.26.28
 frequency 10
ip sla schedule 10 life forever start-time now
ip sla 20
 icmp-echo 44.114.25.25 source-ip 44.114.25.28
 frequency 10
ip sla schedule 20 life forever start-time now
!
route-map VLAN30 permit 10
 match ip address VLAN30
 set ip next-hop verify-availability 44.114.26.26 10 track 10
 set ip next-hop verify-availability 44.114.25.25 20 track 20
!
route-map VLAN31 permit 10
 match ip address VLAN31
 set ip next-hop verify-availability 44.114.25.25 10 track 20
 set ip next-hop verify-availability 44.114.26.26 20 track 10
!
route-map VLAN111 deny 10
 match ip address VLAN111
!

```

```
R28#sh ip sla statistics
IPSLAs Latest Operation Statistics

IPSLA operation id: 10
        Latest RTT: 1 milliseconds
Latest operation start time: 18:31:18 MSK Wed Aug 23 2023
Latest operation return code: OK
Number of successes: 133
Number of failures: 0
Operation time to live: Forever

IPSLA operation id: 20
        Latest RTT: 1 milliseconds
Latest operation start time: 18:31:23 MSK Wed Aug 23 2023
Latest operation return code: OK
Number of successes: 128
Number of failures: 0
Operation time to live: Forever

```


+ #### Шаг 4: Настройка маршрута по-умолчанию для офиса Лабытнанги

#### R27:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R27
!
!
ip route 0.0.0.0 0.0.0.0 44.115.25.25
!
```


