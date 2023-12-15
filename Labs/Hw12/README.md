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