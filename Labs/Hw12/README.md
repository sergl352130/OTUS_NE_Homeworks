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
clock timezone MSK 0 0
!
interface Loopback0
 description R14 Mgmt
 ip address 10.111.3.14 255.255.255.255
 ip ospf 111 area 0
!
interface Loopback1
 description R14 NAT
 ip address 14.14.14.14 255.255.255.255
 ip ospf 111 area 0
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
 network 10.111.3.14 mask 255.255.255.255
 network 14.14.14.14 mask 255.255.255.255
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
 neighbor 22.111.22.22 route-map R22-Kitorn-OUT out
!
ip nat inside source list R14-PAT-IN interface Loopback1 overload
ip nat inside source static 10.111.2.30 14.14.14.14
!
ip access-list standard R14-PAT-IN
 permit 10.111.0.0 0.0.3.255
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
interface Loopback0
 description R15 Mgmt
 ip address 10.111.3.15 255.255.255.255
 ip ospf 111 area 0
!
interface Loopback1
 description R15 NAT
 ip address 15.15.15.15 255.255.255.255
 ip ospf 111 area 0
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
 network 10.111.3.15 mask 255.255.255.255
 network 15.15.15.15 mask 255.255.255.255
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
 neighbor 33.111.21.21 route-map R21-Lamas-IN in
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip nat inside source list R15-PAT-IN interface Loopback1 overload
ip nat inside source static 10.111.2.34 15.15.15.15
!
ip access-list standard R15-PAT-IN
 permit 10.111.0.0 0.0.3.255
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
```

### NAT

```       
VPC1> ping 22.111.22.22 

*33.111.21.21 icmp_seq=1 ttl=252 time=1.345 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=1.990 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=1.731 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.531 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=1.271 ms (ICMP type:3, code:1, Destination host unreachable)

VPC1> ping 33.111.21.21 

84 bytes from 33.111.21.21 icmp_seq=1 ttl=252 time=1.505 ms
84 bytes from 33.111.21.21 icmp_seq=2 ttl=252 time=1.639 ms
84 bytes from 33.111.21.21 icmp_seq=3 ttl=252 time=2.093 ms
84 bytes from 33.111.21.21 icmp_seq=4 ttl=252 time=1.414 ms
84 bytes from 33.111.21.21 icmp_seq=5 ttl=252 time=1.255 ms

VPC7> ping 22.111.22.22 

84 bytes from 22.111.22.22 icmp_seq=1 ttl=252 time=1.747 ms
84 bytes from 22.111.22.22 icmp_seq=2 ttl=252 time=1.557 ms
84 bytes from 22.111.22.22 icmp_seq=3 ttl=252 time=1.570 ms
84 bytes from 22.111.22.22 icmp_seq=4 ttl=252 time=2.053 ms
84 bytes from 22.111.22.22 icmp_seq=5 ttl=252 time=1.810 ms

VPC7> ping 33.111.21.21 

84 bytes from 33.111.21.21 icmp_seq=1 ttl=252 time=2.564 ms
84 bytes from 33.111.21.21 icmp_seq=2 ttl=252 time=1.667 ms
84 bytes from 33.111.21.21 icmp_seq=3 ttl=252 time=1.528 ms
84 bytes from 33.111.21.21 icmp_seq=4 ttl=252 time=1.152 ms
84 bytes from 33.111.21.21 icmp_seq=5 ttl=252 time=1.232 ms

R19#ping 22.111.22.22 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.111.22.22, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R19#ping 18.18.18.18                   
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 18.18.18.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R19#ping 18.18.18.18 source 10.111.3.19
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 18.18.18.18, timeout is 2 seconds:
Packet sent with a source address of 10.111.3.19 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R19#ping 44.114.26.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 44.114.26.28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R20#ping 18.18.18.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 18.18.18.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R20#ping 18.18.18.18 sou
R20#ping 18.18.18.18 source 10.111.3.20
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 18.18.18.18, timeout is 2 seconds:
Packet sent with a source address of 10.111.3.20 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R20#ping 44.114.26.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 44.114.26.28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R20#


R14#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 14.14.14.14:10    10.111.2.30:10     22.111.22.22:10    22.111.22.22:10
--- 14.14.14.14        10.111.2.30        ---                ---

R15#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 15.15.15.15:32521 10.111.0.6:32521   18.18.18.18:32521  18.18.18.18:32521
icmp 15.15.15.15:32777 10.111.0.6:32777   18.18.18.18:32777  18.18.18.18:32777
icmp 15.15.15.15:33033 10.111.0.6:33033   18.18.18.18:33033  18.18.18.18:33033
icmp 15.15.15.15:33289 10.111.0.6:33289   18.18.18.18:33289  18.18.18.18:33289
icmp 15.15.15.15:33545 10.111.0.6:33545   18.18.18.18:33545  18.18.18.18:33545
icmp 15.15.15.15:36873 10.111.0.6:36873   44.114.26.28:36873 44.114.26.28:36873
icmp 15.15.15.15:37129 10.111.0.6:37129   44.114.26.28:37129 44.114.26.28:37129
icmp 15.15.15.15:37385 10.111.0.6:37385   44.114.26.28:37385 44.114.26.28:37385
icmp 15.15.15.15:37641 10.111.0.6:37641   44.114.26.28:37641 44.114.26.28:37641
icmp 15.15.15.15:37897 10.111.0.6:37897   44.114.26.28:37897 44.114.26.28:37897
icmp 15.15.15.15:34569 10.111.1.6:34569   18.18.18.18:34569  18.18.18.18:34569
icmp 15.15.15.15:34825 10.111.1.6:34825   18.18.18.18:34825  18.18.18.18:34825
icmp 15.15.15.15:35081 10.111.1.6:35081   18.18.18.18:35081  18.18.18.18:35081
icmp 15.15.15.15:35337 10.111.1.6:35337   18.18.18.18:35337  18.18.18.18:35337
icmp 15.15.15.15:35593 10.111.1.6:35593   18.18.18.18:35593  18.18.18.18:35593
icmp 15.15.15.15:38665 10.111.1.6:38665   44.114.26.28:38665 44.114.26.28:38665
icmp 15.15.15.15:38921 10.111.1.6:38921   44.114.26.28:38921 44.114.26.28:38921
icmp 15.15.15.15:39177 10.111.1.6:39177   44.114.26.28:39177 44.114.26.28:39177
icmp 15.15.15.15:39433 10.111.1.6:39433   44.114.26.28:39433 44.114.26.28:39433
icmp 15.15.15.15:39689 10.111.1.6:39689   44.114.26.28:39689 44.114.26.28:39689
icmp 15.15.15.15:11    10.111.2.30:11     18.18.18.18:11     18.18.18.18:11
Pro Inside global      Inside local       Outside local      Outside global
icmp 15.15.15.15:3     10.111.2.34:3      18.18.18.18:3      18.18.18.18:3
icmp 15.15.15.15:4     10.111.2.34:4      44.114.26.28:4     44.114.26.28:4
--- 15.15.15.15        10.111.2.34        ---                ---
icmp 15.15.15.15:12    10.111.3.19:12     18.18.18.18:12     18.18.18.18:12
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
interface Loopback1
 description R18 NAT
 ip address 18.18.18.18 255.255.255.248
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
  network 18.18.18.16 0.0.0.7
  eigrp router-id 10.112.3.18
 exit-address-family
!
router bgp 2042
 bgp router-id 10.112.3.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 10.112.3.18 mask 255.255.255.255
 network 18.18.18.16 mask 255.255.255.248
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip nat pool R18-PAT-OUT 18.18.18.17 18.18.18.21 netmask 255.255.255.248
ip nat inside source list R18-PAT-IN pool R18-PAT-OUT overload
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26
!
ip access-list standard R18-PAT-IN
 permit 10.112.0.0 0.0.3.255
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 6 permit 18.18.18.16/29
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
```

### NAT

```
VPC8> ping 14.14.14.14 

84 bytes from 14.14.14.14 icmp_seq=1 ttl=249 time=1.775 ms
84 bytes from 14.14.14.14 icmp_seq=2 ttl=249 time=1.866 ms
84 bytes from 14.14.14.14 icmp_seq=3 ttl=249 time=2.140 ms
84 bytes from 14.14.14.14 icmp_seq=4 ttl=249 time=2.321 ms
84 bytes from 14.14.14.14 icmp_seq=5 ttl=249 time=1.690 ms

VPC8> ping 15.15.15.15

84 bytes from 15.15.15.15 icmp_seq=1 ttl=249 time=2.502 ms
84 bytes from 15.15.15.15 icmp_seq=2 ttl=249 time=1.674 ms
84 bytes from 15.15.15.15 icmp_seq=3 ttl=249 time=2.214 ms
84 bytes from 15.15.15.15 icmp_seq=4 ttl=249 time=1.682 ms
84 bytes from 15.15.15.15 icmp_seq=5 ttl=249 time=1.771 ms

VPC8> ping 44.114.26.28

84 bytes from 44.114.26.28 icmp_seq=1 ttl=250 time=2.162 ms
84 bytes from 44.114.26.28 icmp_seq=2 ttl=250 time=1.781 ms
84 bytes from 44.114.26.28 icmp_seq=3 ttl=250 time=1.945 ms
84 bytes from 44.114.26.28 icmp_seq=4 ttl=250 time=1.739 ms
84 bytes from 44.114.26.28 icmp_seq=5 ttl=250 time=2.142 ms


VPC> ping 14.14.14.14

84 bytes from 14.14.14.14 icmp_seq=1 ttl=249 time=2.850 ms
84 bytes from 14.14.14.14 icmp_seq=2 ttl=249 time=2.405 ms
84 bytes from 14.14.14.14 icmp_seq=3 ttl=249 time=2.272 ms
84 bytes from 14.14.14.14 icmp_seq=4 ttl=249 time=2.022 ms
84 bytes from 14.14.14.14 icmp_seq=5 ttl=249 time=2.272 ms

VPC> ping 15.15.15.15

84 bytes from 15.15.15.15 icmp_seq=1 ttl=249 time=1.913 ms
84 bytes from 15.15.15.15 icmp_seq=2 ttl=249 time=2.840 ms
84 bytes from 15.15.15.15 icmp_seq=3 ttl=249 time=1.712 ms
84 bytes from 15.15.15.15 icmp_seq=4 ttl=249 time=2.218 ms
84 bytes from 15.15.15.15 icmp_seq=5 ttl=249 time=2.333 ms

VPC> ping 44.114.26.28

84 bytes from 44.114.26.28 icmp_seq=1 ttl=250 time=1.810 ms
84 bytes from 44.114.26.28 icmp_seq=2 ttl=250 time=1.667 ms
84 bytes from 44.114.26.28 icmp_seq=3 ttl=250 time=1.347 ms
84 bytes from 44.114.26.28 icmp_seq=4 ttl=250 time=1.427 ms
84 bytes from 44.114.26.28 icmp_seq=5 ttl=250 time=1.408 ms


R18#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 18.18.18.18:52490 10.112.0.2:52490   44.114.26.28:52490 44.114.26.28:52490
icmp 18.18.18.18:52746 10.112.0.2:52746   44.114.26.28:52746 44.114.26.28:52746
icmp 18.18.18.18:53002 10.112.0.2:53002   44.114.26.28:53002 44.114.26.28:53002
icmp 18.18.18.18:53258 10.112.0.2:53258   44.114.26.28:53258 44.114.26.28:53258
icmp 18.18.18.18:53514 10.112.0.2:53514   44.114.26.28:53514 44.114.26.28:53514
icmp 18.18.18.18:50698 10.112.1.2:50698   14.14.14.14:50698  14.14.14.14:50698
icmp 18.18.18.18:50954 10.112.1.2:50954   14.14.14.14:50954  14.14.14.14:50954
icmp 18.18.18.18:51210 10.112.1.2:51210   14.14.14.14:51210  14.14.14.14:51210
icmp 18.18.18.18:51466 10.112.1.2:51466   14.14.14.14:51466  14.14.14.14:51466
icmp 18.18.18.18:51722 10.112.1.2:51722   14.14.14.14:51722  14.14.14.14:51722
icmp 18.18.18.18:57866 10.112.1.2:57866   15.15.15.15:57866  15.15.15.15:57866
icmp 18.18.18.18:58122 10.112.1.2:58122   15.15.15.15:58122  15.15.15.15:58122
icmp 18.18.18.18:58378 10.112.1.2:58378   15.15.15.15:58378  15.15.15.15:58378
icmp 18.18.18.18:58634 10.112.1.2:58634   15.15.15.15:58634  15.15.15.15:58634
icmp 18.18.18.18:58890 10.112.1.2:58890   15.15.15.15:58890  15.15.15.15:58890
icmp 18.18.18.18:61706 10.112.1.2:61706   44.114.26.28:61706 44.114.26.28:61706
icmp 18.18.18.18:61962 10.112.1.2:61962   44.114.26.28:61962 44.114.26.28:61962
icmp 18.18.18.18:62218 10.112.1.2:62218   44.114.26.28:62218 44.114.26.28:62218
icmp 18.18.18.18:62474 10.112.1.2:62474   44.114.26.28:62474 44.114.26.28:62474
icmp 18.18.18.18:62730 10.112.1.2:62730   44.114.26.28:62730 44.114.26.28:62730
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
ip access-list standard R28-STNAT-IN
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

### NAT

```
VPC30> ping 14.14.14.14

84 bytes from 14.14.14.14 icmp_seq=1 ttl=250 time=1.943 ms
84 bytes from 14.14.14.14 icmp_seq=2 ttl=250 time=1.945 ms
84 bytes from 14.14.14.14 icmp_seq=3 ttl=250 time=2.078 ms
84 bytes from 14.14.14.14 icmp_seq=4 ttl=250 time=2.065 ms
84 bytes from 14.14.14.14 icmp_seq=5 ttl=250 time=2.460 ms

VPC30> ping 15.15.15.15

84 bytes from 15.15.15.15 icmp_seq=1 ttl=250 time=2.246 ms
84 bytes from 15.15.15.15 icmp_seq=2 ttl=250 time=2.688 ms
84 bytes from 15.15.15.15 icmp_seq=3 ttl=250 time=7.169 ms
84 bytes from 15.15.15.15 icmp_seq=4 ttl=250 time=2.330 ms
84 bytes from 15.15.15.15 icmp_seq=5 ttl=250 time=1.823 ms

VPC30> ping 18.18.18.18

84 bytes from 18.18.18.18 icmp_seq=1 ttl=252 time=1.369 ms
84 bytes from 18.18.18.18 icmp_seq=2 ttl=252 time=1.925 ms
84 bytes from 18.18.18.18 icmp_seq=3 ttl=252 time=1.537 ms
84 bytes from 18.18.18.18 icmp_seq=4 ttl=252 time=1.204 ms
84 bytes from 18.18.18.18 icmp_seq=5 ttl=252 time=1.278 ms

VPC31> ping 14.14.14.14

84 bytes from 14.14.14.14 icmp_seq=1 ttl=249 time=2.200 ms
84 bytes from 14.14.14.14 icmp_seq=2 ttl=249 time=2.196 ms
84 bytes from 14.14.14.14 icmp_seq=3 ttl=249 time=2.368 ms
84 bytes from 14.14.14.14 icmp_seq=4 ttl=249 time=2.086 ms
84 bytes from 14.14.14.14 icmp_seq=5 ttl=249 time=2.012 ms

VPC31> ping 15.15.15.15

84 bytes from 15.15.15.15 icmp_seq=1 ttl=249 time=3.254 ms
84 bytes from 15.15.15.15 icmp_seq=2 ttl=249 time=3.051 ms
84 bytes from 15.15.15.15 icmp_seq=3 ttl=249 time=2.768 ms
84 bytes from 15.15.15.15 icmp_seq=4 ttl=249 time=2.834 ms
84 bytes from 15.15.15.15 icmp_seq=5 ttl=249 time=2.335 ms

VPC31> ping 18.18.18.18

84 bytes from 18.18.18.18 icmp_seq=1 ttl=252 time=1.316 ms
84 bytes from 18.18.18.18 icmp_seq=2 ttl=252 time=1.721 ms
84 bytes from 18.18.18.18 icmp_seq=3 ttl=252 time=1.493 ms
84 bytes from 18.18.18.18 icmp_seq=4 ttl=252 time=1.481 ms
84 bytes from 18.18.18.18 icmp_seq=5 ttl=252 time=1.262 ms


R28#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 44.114.26.28:27662 10.111.4.66:27662 14.14.14.14:27662  14.14.14.14:27662
icmp 44.114.26.28:27918 10.111.4.66:27918 14.14.14.14:27918  14.14.14.14:27918
icmp 44.114.26.28:28174 10.111.4.66:28174 14.14.14.14:28174  14.14.14.14:28174
icmp 44.114.26.28:29454 10.111.4.66:29454 15.15.15.15:29454  15.15.15.15:29454
icmp 44.114.26.28:29710 10.111.4.66:29710 15.15.15.15:29710  15.15.15.15:29710
icmp 44.114.26.28:29966 10.111.4.66:29966 15.15.15.15:29966  15.15.15.15:29966
icmp 44.114.26.28:30222 10.111.4.66:30222 15.15.15.15:30222  15.15.15.15:30222
icmp 44.114.26.28:30478 10.111.4.66:30478 15.15.15.15:30478  15.15.15.15:30478
icmp 44.114.26.28:33294 10.111.4.66:33294 18.18.18.18:33294  18.18.18.18:33294
icmp 44.114.26.28:33550 10.111.4.66:33550 18.18.18.18:33550  18.18.18.18:33550
icmp 44.114.26.28:33806 10.111.4.66:33806 18.18.18.18:33806  18.18.18.18:33806
icmp 44.114.26.28:34062 10.111.4.66:34062 18.18.18.18:34062  18.18.18.18:34062
icmp 44.114.26.28:34318 10.111.4.66:34318 18.18.18.18:34318  18.18.18.18:34318
icmp 44.114.25.28:35086 10.111.4.131:35086 14.14.14.14:35086 14.14.14.14:35086
icmp 44.114.25.28:35342 10.111.4.131:35342 14.14.14.14:35342 14.14.14.14:35342
icmp 44.114.25.28:35598 10.111.4.131:35598 14.14.14.14:35598 14.14.14.14:35598
icmp 44.114.25.28:35854 10.111.4.131:35854 14.14.14.14:35854 14.14.14.14:35854
icmp 44.114.25.28:36110 10.111.4.131:36110 14.14.14.14:36110 14.14.14.14:36110
icmp 44.114.25.28:37134 10.111.4.131:37134 15.15.15.15:37134 15.15.15.15:37134
icmp 44.114.25.28:37390 10.111.4.131:37390 15.15.15.15:37390 15.15.15.15:37390
icmp 44.114.25.28:37646 10.111.4.131:37646 15.15.15.15:37646 15.15.15.15:37646
Pro Inside global      Inside local       Outside local      Outside global
icmp 44.114.25.28:37902 10.111.4.131:37902 15.15.15.15:37902 15.15.15.15:37902
icmp 44.114.25.28:38158 10.111.4.131:38158 15.15.15.15:38158 15.15.15.15:38158
icmp 44.114.25.28:39694 10.111.4.131:39694 18.18.18.18:39694 18.18.18.18:39694
icmp 44.114.25.28:39950 10.111.4.131:39950 18.18.18.18:39950 18.18.18.18:39950
icmp 44.114.25.28:40206 10.111.4.131:40206 18.18.18.18:40206 18.18.18.18:40206
icmp 44.114.25.28:40462 10.111.4.131:40462 18.18.18.18:40462 18.18.18.18:40462
icmp 44.114.25.28:40718 10.111.4.131:40718 18.18.18.18:40718 18.18.18.18:40718
```

### DHCP, NTP:

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
ip dhcp excluded-address 10.111.0.1 10.111.0.5
!
ip dhcp pool VPC1
 network 10.111.0.0 255.255.255.0
 domain-name VPC1
 default-router 10.111.0.1 
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
ip dhcp excluded-address 10.111.1.1 10.111.1.5
!
ip dhcp pool VPC7
 network 10.111.1.0 255.255.255.0
 domain-name VPC7
 default-router 10.111.1.1 
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
 ip ospf 111 area 10
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
```

### DHCP

```
R12#show ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.111.0.6          0100.5079.6668.01       Infinite                Automatic

R13#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.111.1.6          0100.5079.6668.07       Infinite                Automatic

VPC1> dhcp
DDORA IP 10.111.0.6/24 GW 10.111.0.1

VPC7> dhcp
DDORA IP 10.111.1.6/24 GW 10.111.1.1
```

### NTP

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