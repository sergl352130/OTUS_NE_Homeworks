# Лабораторная работа 13
+ ## VPN. GRE. DMVPN
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw12/Network_topology.png?raw=true)

## Цели:
+ ### Настроить GRE между офисами Москва и С.-Петербург
+ ### Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги


### Этапы:
+ #### Шаг 1: Настройка GRE между офисами Москва и С.-Петербург
+ #### Шаг 2: Настройка DMVPN между офисами Москва и Чокурдах, Лабытнанги
+ #### Шаг 8: Все узлы в офисах в лабораторной работе должны иметь IP связность 


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
 network 10.111.3.19 mask 255.255.255.255
 network 14.14.14.14 mask 255.255.255.255
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
 neighbor 22.111.22.22 route-map R22-Kitorn-OUT out
!
ip nat pool R15-PAT-OUT 14.14.14.14 14.14.14.14 netmask 255.255.255.0
ip nat inside source list R14-PAT-IN pool R14-PAT-OUT overload
ip route 14.14.14.14 255.255.255.255 Null0
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
 network 10.111.3.15 mask 255.255.255.255
 network 15.15.15.15 mask 255.255.255.255
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
 neighbor 33.111.21.21 route-map R21-Lamas-IN in
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip nat pool R15-PAT-OUT 15.15.15.15 15.15.15.15 netmask 255.255.255.0
ip nat inside source list R15-PAT-IN pool R15-PAT-OUT overload
ip nat inside source static 10.111.3.20 15.15.15.15
ip route 15.15.15.15 255.255.255.255 Null0
!
ip access-list standard R15-PAT-IN
 permit 10.111.0.0 0.0.1.255
!
ntp server 10.111.3.12
ntp server 10.111.3.13
!
```

### NAT

```       
VPC1> ping 22.111.22.22

22.111.22.22 icmp_seq=1 timeout
22.111.22.22 icmp_seq=2 timeout
22.111.22.22 icmp_seq=3 timeout
22.111.22.22 icmp_seq=4 timeout
22.111.22.22 icmp_seq=5 timeout

VPC1> ping 10.112.0.2  

*33.111.21.21 icmp_seq=1 ttl=252 time=1.830 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=1.557 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=1.417 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.303 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=1.376 ms (ICMP type:3, code:1, Destination host unreachable)

VPC7> ping 22.111.22.22

22.111.22.22 icmp_seq=1 timeout
22.111.22.22 icmp_seq=2 timeout
22.111.22.22 icmp_seq=3 timeout
22.111.22.22 icmp_seq=4 timeout
22.111.22.22 icmp_seq=5 timeout

VPC7> ping 10.112.0.2

*33.111.21.21 icmp_seq=1 ttl=252 time=1.920 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=2 ttl=252 time=1.687 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=3 ttl=252 time=2.098 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=4 ttl=252 time=1.557 ms (ICMP type:3, code:1, Destination host unreachable)
*33.111.21.21 icmp_seq=5 ttl=252 time=1.670 ms (ICMP type:3, code:1, Destination host unreachable)

R19#ping 10.112.3.18                   
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.112.3.18, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R19#ping 10.112.3.18 source 10.111.3.19
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.112.3.18, timeout is 2 seconds:
Packet sent with a source address of 10.111.3.19 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R18#ping 10.111.3.19
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.111.3.19, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R18#ping 10.111.3.19 source 10.112.3.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.111.3.19, timeout is 2 seconds:
Packet sent with a source address of 10.112.3.18 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms

R28#ping 10.111.3.19
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.111.3.19, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R28#ping 10.111.3.19 source 10.111.4.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.111.3.19, timeout is 2 seconds:
Packet sent with a source address of 10.111.4.28 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R20#ping 10.112.3.18                   
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.112.3.18, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R20#ping 10.112.3.18 source 10.111.3.20
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.112.3.18, timeout is 2 seconds:
Packet sent with a source address of 10.111.3.20 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms


R14#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 14.14.14.14:3211  10.111.0.6:3211    22.111.22.22:3211  22.111.22.22:3211
icmp 14.14.14.14:3723  10.111.0.6:3723    22.111.22.22:3723  22.111.22.22:3723
icmp 14.14.14.14:4235  10.111.0.6:4235    22.111.22.22:4235  22.111.22.22:4235
icmp 14.14.14.14:1024  10.111.0.6:4747    22.111.22.22:4747  22.111.22.22:1024
icmp 14.14.14.14:1025  10.111.0.6:5259    22.111.22.22:5259  22.111.22.22:1025
icmp 14.14.14.14:4747  10.111.1.6:4747    22.111.22.22:4747  22.111.22.22:4747
icmp 14.14.14.14:5259  10.111.1.6:5259    22.111.22.22:5259  22.111.22.22:5259
icmp 14.14.14.14:5771  10.111.1.6:5771    22.111.22.22:5771  22.111.22.22:5771
icmp 14.14.14.14:6283  10.111.1.6:6283    22.111.22.22:6283  22.111.22.22:6283
icmp 14.14.14.14:6795  10.111.1.6:6795    22.111.22.22:6795  22.111.22.22:6795

R15#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 15.15.15.15:20109 10.111.0.6:20109   10.112.0.2:20109   10.112.0.2:20109
icmp 15.15.15.15:20365 10.111.0.6:20365   10.112.0.2:20365   10.112.0.2:20365
icmp 15.15.15.15:20621 10.111.0.6:20621   10.112.0.2:20621   10.112.0.2:20621
icmp 15.15.15.15:20877 10.111.0.6:20877   10.112.0.2:20877   10.112.0.2:20877
icmp 15.15.15.15:21133 10.111.0.6:21133   10.112.0.2:21133   10.112.0.2:21133
icmp 15.15.15.15:21389 10.111.1.6:21389   10.112.0.2:21389   10.112.0.2:21389
icmp 15.15.15.15:21645 10.111.1.6:21645   10.112.0.2:21645   10.112.0.2:21645
icmp 15.15.15.15:21901 10.111.1.6:21901   10.112.0.2:21901   10.112.0.2:21901
icmp 15.15.15.15:22157 10.111.1.6:22157   10.112.0.2:22157   10.112.0.2:22157
icmp 15.15.15.15:22413 10.111.1.6:22413   10.112.0.2:22413   10.112.0.2:22413
icmp 15.15.15.15:11    10.111.3.20:11     10.112.3.18:11     10.112.3.18:11
--- 15.15.15.15        10.111.3.20        ---                ---
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
 permit 10.112.0.0 0.0.1.255
 permit 10.112.2.0 0.0.0.255
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 6 permit 18.18.18.16/29
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
```

### NAT

```
VPC8> ping 10.111.1.6

*44.112.26.26 icmp_seq=1 ttl=252 time=0.932 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.26.26 icmp_seq=2 ttl=252 time=1.081 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.26.26 icmp_seq=3 ttl=252 time=1.269 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.26.26 icmp_seq=4 ttl=252 time=1.470 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.26.26 icmp_seq=5 ttl=252 time=1.013 ms (ICMP type:3, code:1, Destination host unreachable)


VPC> ping 10.111.1.6

*44.112.24.24 icmp_seq=1 ttl=252 time=0.887 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.24.24 icmp_seq=2 ttl=252 time=1.766 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.24.24 icmp_seq=3 ttl=252 time=1.142 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.24.24 icmp_seq=4 ttl=252 time=1.312 ms (ICMP type:3, code:1, Destination host unreachable)
*44.112.24.24 icmp_seq=5 ttl=252 time=1.442 ms (ICMP type:3, code:1, Destination host unreachable)


R18#sh ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 18.18.18.18:54400 10.112.0.2:54400   10.111.1.6:54400   10.111.1.6:54400
icmp 18.18.18.18:54656 10.112.0.2:54656   10.111.1.6:54656   10.111.1.6:54656
icmp 18.18.18.18:54912 10.112.0.2:54912   10.111.1.6:54912   10.111.1.6:54912
icmp 18.18.18.18:1024  10.112.0.2:55168   10.111.1.6:55168   10.111.1.6:1024
icmp 18.18.18.18:1025  10.112.0.2:55424   10.111.1.6:55424   10.111.1.6:1025
icmp 18.18.18.18:55168 10.112.1.2:55168   10.111.1.6:55168   10.111.1.6:55168
icmp 18.18.18.18:55424 10.112.1.2:55424   10.111.1.6:55424   10.111.1.6:55424
icmp 18.18.18.18:55680 10.112.1.2:55680   10.111.1.6:55680   10.111.1.6:55680
icmp 18.18.18.18:55936 10.112.1.2:55936   10.111.1.6:55936   10.111.1.6:55936
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
VPC30> ping 10.111.1.6

*44.114.26.26 icmp_seq=1 ttl=254 time=0.811 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.26.26 icmp_seq=2 ttl=254 time=1.152 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.26.26 icmp_seq=3 ttl=254 time=0.929 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.26.26 icmp_seq=4 ttl=254 time=1.492 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.26.26 icmp_seq=5 ttl=254 time=1.063 ms (ICMP type:3, code:1, Destination host unreachable)

VPC31> ping 10.111.1.6

*44.114.25.25 icmp_seq=1 ttl=254 time=0.793 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.25.25 icmp_seq=2 ttl=254 time=1.165 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.25.25 icmp_seq=3 ttl=254 time=1.260 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.25.25 icmp_seq=4 ttl=254 time=1.116 ms (ICMP type:3, code:1, Destination host unreachable)
*44.114.25.25 icmp_seq=5 ttl=254 time=1.123 ms (ICMP type:3, code:1, Destination host unreachable)


R28#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 44.114.26.28:49553 10.111.4.66:49553 10.111.1.6:49553   10.111.1.6:49553
icmp 44.114.26.28:49809 10.111.4.66:49809 10.111.1.6:49809   10.111.1.6:49809
icmp 44.114.26.28:50065 10.111.4.66:50065 10.111.1.6:50065   10.111.1.6:50065
icmp 44.114.26.28:50321 10.111.4.66:50321 10.111.1.6:50321   10.111.1.6:50321
icmp 44.114.26.28:50577 10.111.4.66:50577 10.111.1.6:50577   10.111.1.6:50577
icmp 44.114.25.28:51089 10.111.4.131:51089 10.111.1.6:51089  10.111.1.6:51089
icmp 44.114.25.28:51345 10.111.4.131:51345 10.111.1.6:51345  10.111.1.6:51345
icmp 44.114.25.28:51601 10.111.4.131:51601 10.111.1.6:51601  10.111.1.6:51601
icmp 44.114.25.28:51857 10.111.4.131:51857 10.111.1.6:51857  10.111.1.6:51857
icmp 44.114.25.28:52113 10.111.4.131:52113 10.111.1.6:52113  10.111.1.6:52113
```