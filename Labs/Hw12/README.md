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


### NAT

#### R14:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
clock timezone MSK 0 0
!
interface Ethernet0/0
 description p2p to R12
 ip address 10.111.2.10 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/1
 description p2p to R13
 ip address 10.111.2.22 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/2
 description p2p to R15
 ip address 10.111.2.25 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/3
 description p2p to R19
 ip address 10.111.2.29 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 101
!
interface Ethernet1/0
 description to R22 Kitorn
 ip address 22.111.22.14 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!
router ospf 111
 router-id 10.111.3.14
 area 101 stub no-summary
 default-information originate always
!
router bgp 1001
 bgp router-id 10.111.3.14
 bgp log-neighbor-changes
 network 10.111.3.19 mask 255.255.255.255
 network 40.40.40.40 mask 255.255.255.255
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
 neighbor 22.111.22.22 route-map R22-Kitorn-OUT out
!
ip nat pool R14-PAT-OUT 40.40.40.40 40.40.40.40 netmask 255.255.255.0
ip nat inside source list R14-PAT-IN pool R14-PAT-OUT overload
ip nat inside source static 10.111.3.20 40.40.40.40
ip route 40.40.40.40 255.255.255.255 Null0
!
ip access-list standard R14-PAT-IN
 permit 10.111.0.0 0.0.1.255
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
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
clock timezone MSK 0 0
!
interface Ethernet0/0
 description p2p to R13
 ip address 10.111.2.18 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/1
 description p2p to R12
 ip address 10.111.2.14 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/2
 description p2p to R14
 ip address 10.111.2.26 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
!
interface Ethernet0/3
 description p2p to R20
 ip address 10.111.2.33 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 102
!
interface Ethernet1/0
 description to R21 Lamas
 ip address 33.111.21.15 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!
router ospf 111
 router-id 10.111.3.15
 area 102 filter-list prefix NOarea101 in
 default-information originate always
!
router bgp 1001
 bgp router-id 10.111.3.15
 bgp log-neighbor-changes
 network 50.50.50.50 mask 255.255.255.255
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
 neighbor 33.111.21.21 route-map R21-Lamas-IN in
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip nat pool R15-PAT-OUT 50.50.50.50 50.50.50.50 netmask 255.255.255.0
ip nat inside source list R15-PAT-IN pool R15-PAT-OUT overload
ip nat inside source static 10.111.3.20 50.50.50.50
ip route 50.50.50.50 255.255.255.255 Null0
!
ip access-list standard R15-PAT-IN
 permit 10.111.0.0 0.0.1.255
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
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
interface Ethernet0/0
 description p2p to R17
 ip address 10.112.2.18 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 description p2p to R16
 ip address 10.112.2.22 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet1/0
 description to R24 Triada
 ip address 44.112.24.18 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet1/1
 description to R26 Triada
 ip address 44.112.26.18 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!
router bgp 2042
 bgp router-id 10.112.3.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 80.80.80.16 mask 255.255.255.248
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip nat pool R18-PAT-OUT 80.80.80.17 80.80.80.21 netmask 255.255.255.248
ip nat inside source list R18-PAT-IN pool R18-PAT-OUT overload
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26
ip route 80.80.80.16 255.255.255.248 Null0
!
ip access-list standard R18-PAT-IN
 permit 10.112.0.0 0.0.1.255
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 7 permit 80.80.80.16/29
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
```

#### R28:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R28
!
interface Loopback0
 ip address 10.111.4.28 255.255.255.255
!
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.30
 description LAN VPC30
 encapsulation dot1Q 30
 ip address 10.111.4.65 255.255.255.192
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map VLAN30
!
interface Ethernet0/0.31
 description LAN VPC31
 encapsulation dot1Q 31
 ip address 10.111.4.129 255.255.255.192
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map VLAN31
!
interface Ethernet1/0
 description to R26 Triada
 ip address 44.114.26.28 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!         
interface Ethernet1/1
 description to R25 Triada
 ip address 44.114.25.28 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!
ip nat inside source route-map RM-R28-R25-NAT interface Ethernet1/1 overload
ip nat inside source route-map RM-R28-R26-NAT interface Ethernet1/0 overload
ip route 0.0.0.0 0.0.0.0 44.114.25.25
ip route 0.0.0.0 0.0.0.0 44.114.26.26
!
ip access-list standard R28-NAT-IN
 permit 10.111.4.0 0.0.0.255
!
route-map RM-R28-R25-NAT permit 10
 match ip address R28-NAT-IN
 match interface Ethernet1/1
!
route-map RM-R28-R26-NAT permit 10
 match ip address R28-NAT-IN
 match interface Ethernet1/0
!
```

### Проверка NAT

```
VPC1> ping 10.112.0.2  

*33.111.21.21 icmp_seq=1 ttl=252 time=1.419 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=1.611 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=1.743 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.938 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=1.897 ms (ICMP type:3, code:1, Destination host unreachable)

VPC1> ping 10.112.1.2 

*33.111.21.21 icmp_seq=1 ttl=252 time=2.108 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=1.425 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=1.389 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.955 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=2.158 ms (ICMP type:3, code:1, Destination host unreachable)

VPC7> ping 10.112.0.2

*33.111.21.21 icmp_seq=1 ttl=252 time=1.889 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=1.691 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=2.243 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.708 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=1.291 ms (ICMP type:3, code:1, Destination host unreachable)

VPC7> ping 10.112.1.2  

*33.111.21.21 icmp_seq=1 ttl=252 time=2.065 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=2.163 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=1.968 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.709 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=2.275 ms (ICMP type:3, code:1, Destination host unreachable)

R20#ping 10.112.0.2 source 10.111.3.20
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.112.0.2, timeout is 2 seconds:
Packet sent with a source address of 10.111.3.20 
U.U.U
Success rate is 0 percent (0/5)


VPC8> ping 10.111.3.19

84 bytes from 10.111.3.19 icmp_seq=1 ttl=248 time=2.540 ms
84 bytes from 10.111.3.19 icmp_seq=2 ttl=248 time=2.047 ms
84 bytes from 10.111.3.19 icmp_seq=3 ttl=248 time=2.301 ms
84 bytes from 10.111.3.19 icmp_seq=4 ttl=248 time=2.350 ms
84 bytes from 10.111.3.19 icmp_seq=5 ttl=248 time=2.485 ms

VPC> ping 10.111.3.19

84 bytes from 10.111.3.19 icmp_seq=1 ttl=248 time=1.972 ms
84 bytes from 10.111.3.19 icmp_seq=2 ttl=248 time=2.857 ms
84 bytes from 10.111.3.19 icmp_seq=3 ttl=248 time=2.824 ms
84 bytes from 10.111.3.19 icmp_seq=4 ttl=248 time=2.013 ms
84 bytes from 10.111.3.19 icmp_seq=5 ttl=248 time=2.337 ms


VPC30> ping 10.111.3.19

84 bytes from 10.111.3.19 icmp_seq=1 ttl=249 time=2.140 ms
84 bytes from 10.111.3.19 icmp_seq=2 ttl=249 time=2.628 ms
84 bytes from 10.111.3.19 icmp_seq=3 ttl=249 time=2.607 ms
84 bytes from 10.111.3.19 icmp_seq=4 ttl=249 time=2.326 ms
84 bytes from 10.111.3.19 icmp_seq=5 ttl=249 time=2.200 ms

VPC31> ping 10.111.3.19

84 bytes from 10.111.3.19 icmp_seq=1 ttl=248 time=2.776 ms
84 bytes from 10.111.3.19 icmp_seq=2 ttl=248 time=2.691 ms
84 bytes from 10.111.3.19 icmp_seq=3 ttl=248 time=2.438 ms
84 bytes from 10.111.3.19 icmp_seq=4 ttl=248 time=2.353 ms
84 bytes from 10.111.3.19 icmp_seq=5 ttl=248 time=2.272 ms


R15#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 50.50.50.50:36994 10.111.0.2:36994   10.112.0.2:36994   10.112.0.2:36994
icmp 50.50.50.50:37250 10.111.0.2:37250   10.112.0.2:37250   10.112.0.2:37250
icmp 50.50.50.50:37506 10.111.0.2:37506   10.112.0.2:37506   10.112.0.2:37506
icmp 50.50.50.50:37762 10.111.0.2:37762   10.112.0.2:37762   10.112.0.2:37762
icmp 50.50.50.50:38018 10.111.0.2:38018   10.112.0.2:38018   10.112.0.2:38018
icmp 50.50.50.50:39810 10.111.0.2:39810   10.112.1.2:39810   10.112.1.2:39810
icmp 50.50.50.50:40066 10.111.0.2:40066   10.112.1.2:40066   10.112.1.2:40066
icmp 50.50.50.50:40322 10.111.0.2:40322   10.112.1.2:40322   10.112.1.2:40322
icmp 50.50.50.50:40578 10.111.0.2:40578   10.112.1.2:40578   10.112.1.2:40578
icmp 50.50.50.50:40834 10.111.0.2:40834   10.112.1.2:40834   10.112.1.2:40834
icmp 50.50.50.50:1024  10.111.1.2:38018   10.112.0.2:38018   10.112.0.2:1024
icmp 50.50.50.50:38274 10.111.1.2:38274   10.112.0.2:38274   10.112.0.2:38274
icmp 50.50.50.50:38530 10.111.1.2:38530   10.112.0.2:38530   10.112.0.2:38530
icmp 50.50.50.50:38786 10.111.1.2:38786   10.112.0.2:38786   10.112.0.2:38786
icmp 50.50.50.50:39042 10.111.1.2:39042   10.112.0.2:39042   10.112.0.2:39042
icmp 50.50.50.50:42626 10.111.1.2:42626   10.112.1.2:42626   10.112.1.2:42626
icmp 50.50.50.50:42882 10.111.1.2:42882   10.112.1.2:42882   10.112.1.2:42882
icmp 50.50.50.50:43138 10.111.1.2:43138   10.112.1.2:43138   10.112.1.2:43138
icmp 50.50.50.50:43394 10.111.1.2:43394   10.112.1.2:43394   10.112.1.2:43394
icmp 50.50.50.50:43650 10.111.1.2:43650   10.112.1.2:43650   10.112.1.2:43650
icmp 50.50.50.50:4     10.111.3.20:4      10.112.0.2:4       10.112.0.2:4
Pro Inside global      Inside local       Outside local      Outside global
--- 50.50.50.50        10.111.3.20        ---                ---

R18#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 80.80.80.17:7044  10.112.0.2:7044    10.111.3.19:7044   10.111.3.19:7044
icmp 80.80.80.17:7300  10.112.0.2:7300    10.111.3.19:7300   10.111.3.19:7300
icmp 80.80.80.17:7556  10.112.0.2:7556    10.111.3.19:7556   10.111.3.19:7556
icmp 80.80.80.17:7812  10.112.0.2:7812    10.111.3.19:7812   10.111.3.19:7812
icmp 80.80.80.17:8068  10.112.0.2:8068    10.111.3.19:8068   10.111.3.19:8068
icmp 80.80.80.17:8580  10.112.1.2:8580    10.111.3.19:8580   10.111.3.19:8580
icmp 80.80.80.17:8836  10.112.1.2:8836    10.111.3.19:8836   10.111.3.19:8836
icmp 80.80.80.17:9092  10.112.1.2:9092    10.111.3.19:9092   10.111.3.19:9092
icmp 80.80.80.17:9348  10.112.1.2:9348    10.111.3.19:9348   10.111.3.19:9348
icmp 80.80.80.17:9604  10.112.1.2:9604    10.111.3.19:9604   10.111.3.19:9604

R28#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 44.114.26.28:51332 10.111.4.66:51332 10.111.3.19:51332  10.111.3.19:51332
icmp 44.114.26.28:51588 10.111.4.66:51588 10.111.3.19:51588  10.111.3.19:51588
icmp 44.114.26.28:51844 10.111.4.66:51844 10.111.3.19:51844  10.111.3.19:51844
icmp 44.114.26.28:52100 10.111.4.66:52100 10.111.3.19:52100  10.111.3.19:52100
icmp 44.114.26.28:52356 10.111.4.66:52356 10.111.3.19:52356  10.111.3.19:52356
icmp 44.114.25.28:53380 10.111.4.131:53380 10.111.3.19:53380 10.111.3.19:53380
icmp 44.114.25.28:53636 10.111.4.131:53636 10.111.3.19:53636 10.111.3.19:53636
icmp 44.114.25.28:53892 10.111.4.131:53892 10.111.3.19:53892 10.111.3.19:53892
icmp 44.114.25.28:54148 10.111.4.131:54148 10.111.3.19:54148 10.111.3.19:54148
icmp 44.114.25.28:54404 10.111.4.131:54404 10.111.3.19:54404 10.111.3.19:54404
```

### Проверка NAT через R14

```
R15(config)#interface Ethernet1/0
R15(config-if)#shutdown
R15(config-if)#
Dec 28 14:29:40.016: %BGP-5-NBR_RESET: Neighbor 33.111.21.21 reset (Interface flap)
Dec 28 14:29:40.017: %BGP-5-ADJCHANGE: neighbor 33.111.21.21 Down Interface flap
Dec 28 14:29:40.017: %BGP_SESSION-5-ADJCHANGE: neighbor 33.111.21.21 IPv4 Unicast topology base removed from session  Interface flap
R15(config-if)#
Dec 28 14:29:42.012: %LINK-5-CHANGED: Interface Ethernet1/0, changed state to administratively down
Dec 28 14:29:43.017: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/0, changed state to down
R15(config-if)#

VPC1> ping 10.112.0.2

10.112.0.2 icmp_seq=1 timeout
10.112.0.2 icmp_seq=2 timeout
10.112.0.2 icmp_seq=3 timeout
10.112.0.2 icmp_seq=4 timeout
10.112.0.2 icmp_seq=5 timeout

VPC1> ping 10.112.1.2

10.112.1.2 icmp_seq=1 timeout
10.112.1.2 icmp_seq=2 timeout
10.112.1.2 icmp_seq=3 timeout
10.112.1.2 icmp_seq=4 timeout
10.112.1.2 icmp_seq=5 timeout

VPC7> ping 10.112.0.2

10.112.0.2 icmp_seq=1 timeout
10.112.0.2 icmp_seq=2 timeout
10.112.0.2 icmp_seq=3 timeout
10.112.0.2 icmp_seq=4 timeout
10.112.0.2 icmp_seq=5 timeout

VPC7> ping 10.112.1.2

10.112.1.2 icmp_seq=1 timeout
10.112.1.2 icmp_seq=2 timeout
10.112.1.2 icmp_seq=3 timeout
10.112.1.2 icmp_seq=4 timeout
10.112.1.2 icmp_seq=5 timeout

R20#ping 10.112.0.2 source 10.111.3.20
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.112.0.2, timeout is 2 seconds:
Packet sent with a source address of 10.111.3.20 
.....
Success rate is 0 percent (0/5)


R14#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 40.40.40.40:40070 10.111.0.2:40070   10.112.0.2:40070   10.112.0.2:40070
icmp 40.40.40.40:40582 10.111.0.2:40582   10.112.0.2:40582   10.112.0.2:40582
icmp 40.40.40.40:41094 10.111.0.2:41094   10.112.0.2:41094   10.112.0.2:41094
icmp 40.40.40.40:42118 10.111.0.2:42118   10.112.1.2:42118   10.112.1.2:42118
icmp 40.40.40.40:42630 10.111.0.2:42630   10.112.1.2:42630   10.112.1.2:42630
icmp 40.40.40.40:43142 10.111.0.2:43142   10.112.1.2:43142   10.112.1.2:43142
icmp 40.40.40.40:43654 10.111.0.2:43654   10.112.1.2:43654   10.112.1.2:43654
icmp 40.40.40.40:1024  10.111.0.2:44166   10.112.1.2:44166   10.112.1.2:1024
icmp 40.40.40.40:44166 10.111.1.2:44166   10.112.0.2:44166   10.112.0.2:44166
icmp 40.40.40.40:44678 10.111.1.2:44678   10.112.0.2:44678   10.112.0.2:44678
icmp 40.40.40.40:45190 10.111.1.2:45190   10.112.0.2:45190   10.112.0.2:45190
icmp 40.40.40.40:45702 10.111.1.2:45702   10.112.0.2:45702   10.112.0.2:45702
icmp 40.40.40.40:46214 10.111.1.2:46214   10.112.0.2:46214   10.112.0.2:46214
icmp 40.40.40.40:47238 10.111.1.2:47238   10.112.1.2:47238   10.112.1.2:47238
icmp 40.40.40.40:47750 10.111.1.2:47750   10.112.1.2:47750   10.112.1.2:47750
icmp 40.40.40.40:48262 10.111.1.2:48262   10.112.1.2:48262   10.112.1.2:48262
icmp 40.40.40.40:48774 10.111.1.2:48774   10.112.1.2:48774   10.112.1.2:48774
icmp 40.40.40.40:49286 10.111.1.2:49286   10.112.1.2:49286   10.112.1.2:49286
icmp 40.40.40.40:5     10.111.3.20:5      10.112.0.2:5       10.112.0.2:5
--- 40.40.40.40        10.111.3.20        ---                ---
```

### DHCP, NTP

#### R12:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R12
!
clock timezone MSK 0 0
clock calendar-valid
!
ip dhcp excluded-address 10.111.0.129 10.111.0.254
ip dhcp excluded-address 10.111.1.1 10.111.1.128
!
ip dhcp pool VPC1
 network 10.111.0.0 255.255.255.0
 domain-name VPC1
 default-router 10.111.0.1 
 lease infinite
!
ip dhcp pool VPC7
 network 10.111.1.0 255.255.255.0
 domain-name VPC7
 default-router 10.111.1.1 
 lease infinite
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
interface Ethernet1/0
 description p2p to SW4
 ip address 10.111.2.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 10
!
ntp master 4
ntp update-calendar
!
```

#### R13:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R13
!
clock timezone MSK 0 0
clock calendar-valid
!
ip dhcp excluded-address 10.111.1.129 10.111.1.254
ip dhcp excluded-address 10.111.0.1 10.111.0.128
!
ip dhcp pool VPC7
 network 10.111.1.0 255.255.255.0
 domain-name VPC7
 default-router 10.111.1.1 
 lease infinite
!
ip dhcp pool VPC1
 network 10.111.0.0 255.255.255.0
 domain-name VPC1
 default-router 10.111.0.1 
 lease infinite
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
interface Ethernet1/0
 description p2p to SW5
 ip address 10.111.2.6 255.255.255.252
 ip ospf network point-to-point
 ip ospf 111 area 10
!
ntp master 5
ntp update-calendar
!
```

#### SW4:

```
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW4
!
clock timezone MSK 0 0
!
interface Vlan11
 description Default gateway VLAN11
 ip address 10.111.0.1 255.255.255.0
 ip helper-address 10.111.2.2
 ip helper-address 10.111.2.6
 ip ospf 111 area 10
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
```

#### SW5:

```
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW5
!
clock timezone MSK 0 0
!
interface Vlan17
 description Default gateway VLAN17
 ip address 10.111.1.1 255.255.255.0
 ip helper-address 10.111.2.6
 ip helper-address 10.111.2.2
 ip ospf 111 area 10
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
```

### Проверка DHCP

```
R12#show ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.111.0.2          0100.5079.6668.01       Infinite                Automatic

R13#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.111.1.2          0100.5079.6668.07       Infinite                Automatic

VPC1> dhcp
DDORA IP 10.111.0.2/24 GW 10.111.0.1

VPC7> dhcp
DDORA IP 10.111.1.2/24 GW 10.111.1.1
```

### Проверка NTP

```
R12#show clock detail 
19:48:33.852 MSK Mon Dec 11 2023
Time source is NTP
R12#
R12#show ntp associations 

  address         ref clock       st   when   poll reach  delay  offset   disp
*~127.127.1.1     .LOCL.           3      1     16   377  0.000   0.000  1.204
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 R13#show clock detail
19:50:20.146 MSK Mon Dec 11 2023
Time source is NTP
R13#
R13#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
*~127.127.1.1     .LOCL.           4      7     16   377  0.000   0.000  1.204
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 SW4#show clock detail
19:51:39.013 MSK Mon Dec 11 2023
Time source is NTP
SW4#
SW4#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
+~10.111.3.12     127.127.1.1      4    400   1024   377  0.000  -1.000  2.007
*~10.111.3.13     127.127.1.1      5    370   1024   377  1.000   0.500  1.987
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 SW5#show clock detail
19:53:07.483 MSK Mon Dec 11 2023
Time source is NTP
SW5#
SW5#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
+~10.111.3.12     127.127.1.1      4    758   1024   377  0.000  -1.000  1.975
*~10.111.3.13     127.127.1.1      5    491   1024   377  0.000   0.000  1.989
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 SW2#show clock detail
*11:30:30.055 MSK Tue Dec 12 2023
Time source is NTP
SW2#
SW2#sho ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.111.3.12     127.127.1.1      4     37     64     1  2.000  -1.000 189.44
+~10.111.3.13     127.127.1.1      5     35     64     1  1.000  -0.500 189.44
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 SW2#show clock detail
*11:30:30.055 MSK Tue Dec 12 2023
Time source is NTP
SW2#
SW2#sho ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.111.3.12     127.127.1.1      4     37     64     1  2.000  -1.000 189.44
+~10.111.3.13     127.127.1.1      5     35     64     1  1.000  -0.500 189.44
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 R19#show clock detail
11:33:29.954 MSK Tue Dec 12 2023
Time source is NTP
R19#
R19#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
+~10.111.3.12     127.127.1.1      4    208    512   377  0.000  -1.000  2.047
*~10.111.3.13     127.127.1.1      5      1    512   377  0.000   0.000  2.183
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

 R20#show clock detail
11:33:29.954 MSK Tue Dec 12 2023
Time source is NTP
R20#
R20#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.111.3.12     127.127.1.1      4    400   1024   377  0.000  -1.000  2.076
+~10.111.3.13     127.127.1.1      5    482    512   377  1.000  -0.500  1.974
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
 ```