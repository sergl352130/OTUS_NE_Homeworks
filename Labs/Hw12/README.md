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
ip nat inside source list R14-PAT-IN interface Ethernet1/0 overload
ip nat inside source static 10.111.2.30 22.111.22.19
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
ip nat inside source list R15-PAT-IN interface Ethernet1/0 overload
ip nat inside source static 10.111.2.34 33.111.21.20
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
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R19#ping 33.111.21.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 33.111.21.21, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R20#ping 22.111.22.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.111.22.22, timeout is 2 seconds:
U.U.U
Success rate is 0 percent (0/5)
R20#ping 33.111.21.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 33.111.21.21, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms


R14#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 22.111.22.14:51113 10.111.1.6:51113  22.111.22.22:51113 22.111.22.22:51113
icmp 22.111.22.14:51369 10.111.1.6:51369  22.111.22.22:51369 22.111.22.22:51369
icmp 22.111.22.14:51625 10.111.1.6:51625  22.111.22.22:51625 22.111.22.22:51625
icmp 22.111.22.14:51881 10.111.1.6:51881  22.111.22.22:51881 22.111.22.22:51881
icmp 22.111.22.14:52137 10.111.1.6:52137  22.111.22.22:52137 22.111.22.22:52137
icmp 22.111.22.19:3    10.111.2.30:3      22.111.22.22:3     22.111.22.22:3
--- 22.111.22.19       10.111.2.30        ---                ---

R15#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 33.111.21.15:36779 10.111.0.6:36779  22.111.22.22:36779 22.111.22.22:36779
icmp 33.111.21.15:37035 10.111.0.6:37035  22.111.22.22:37035 22.111.22.22:37035
icmp 33.111.21.15:37291 10.111.0.6:37291  22.111.22.22:37291 22.111.22.22:37291
icmp 33.111.21.15:37547 10.111.0.6:37547  22.111.22.22:37547 22.111.22.22:37547
icmp 33.111.21.15:37803 10.111.0.6:37803  22.111.22.22:37803 22.111.22.22:37803
icmp 33.111.21.15:41899 10.111.0.6:41899  33.111.21.21:41899 33.111.21.21:41899
icmp 33.111.21.15:42155 10.111.0.6:42155  33.111.21.21:42155 33.111.21.21:42155
icmp 33.111.21.15:42411 10.111.0.6:42411  33.111.21.21:42411 33.111.21.21:42411
icmp 33.111.21.15:42667 10.111.0.6:42667  33.111.21.21:42667 33.111.21.21:42667
icmp 33.111.21.15:42923 10.111.0.6:42923  33.111.21.21:42923 33.111.21.21:42923
icmp 33.111.21.15:43179 10.111.1.6:43179  33.111.21.21:43179 33.111.21.21:43179
icmp 33.111.21.15:43435 10.111.1.6:43435  33.111.21.21:43435 33.111.21.21:43435
icmp 33.111.21.15:43691 10.111.1.6:43691  33.111.21.21:43691 33.111.21.21:43691
icmp 33.111.21.15:43947 10.111.1.6:43947  33.111.21.21:43947 33.111.21.21:43947
icmp 33.111.21.15:44203 10.111.1.6:44203  33.111.21.21:44203 33.111.21.21:44203
icmp 33.111.21.15:7    10.111.2.30:7      33.111.21.21:7     33.111.21.21:7
icmp 33.111.21.20:4    10.111.2.34:4      22.111.22.22:4     22.111.22.22:4
icmp 33.111.21.20:5    10.111.2.34:5      33.111.21.21:5     33.111.21.21:5
--- 33.111.21.20       10.111.2.34        ---                ---
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
ip nat pool R18-R24-PAT-OUT 44.112.24.250 44.112.24.254 netmask 255.255.255.0
ip nat pool R18-R26-PAT-OUT 44.112.26.250 44.112.26.254 netmask 255.255.255.0
ip nat inside source route-map RM-R18-R24-PAT pool R18-R24-PAT-OUT overload
ip nat inside source route-map RM-R18-R26-PAT pool R18-R26-PAT-OUT overload
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26
!
ip access-list standard R18-PAT-IN
 permit 10.112.0.0 0.0.3.255
!
route-map RM-R18-R24-PAT permit 10
 match ip address R18-PAT-IN
 match interface Ethernet1/0
!         
route-map RM-R18-R26-PAT permit 10
 match ip address R18-PAT-IN
 match interface Ethernet1/1
!
```

### NAT

```
VPC8> ping 44.112.24.24

84 bytes from 44.112.24.24 icmp_seq=1 ttl=252 time=1.305 ms
84 bytes from 44.112.24.24 icmp_seq=2 ttl=252 time=1.241 ms
84 bytes from 44.112.24.24 icmp_seq=3 ttl=252 time=2.089 ms
84 bytes from 44.112.24.24 icmp_seq=4 ttl=252 time=0.995 ms
84 bytes from 44.112.24.24 icmp_seq=5 ttl=252 time=1.680 ms

VPC8> ping 44.112.26.26

84 bytes from 44.112.26.26 icmp_seq=1 ttl=252 time=1.209 ms
84 bytes from 44.112.26.26 icmp_seq=2 ttl=252 time=1.138 ms
84 bytes from 44.112.26.26 icmp_seq=3 ttl=252 time=1.367 ms
84 bytes from 44.112.26.26 icmp_seq=4 ttl=252 time=1.140 ms
84 bytes from 44.112.26.26 icmp_seq=5 ttl=252 time=1.337 ms


VPC> ping 44.112.24.24

84 bytes from 44.112.24.24 icmp_seq=1 ttl=252 time=1.352 ms
84 bytes from 44.112.24.24 icmp_seq=2 ttl=252 time=1.830 ms
84 bytes from 44.112.24.24 icmp_seq=3 ttl=252 time=1.538 ms
84 bytes from 44.112.24.24 icmp_seq=4 ttl=252 time=1.197 ms
84 bytes from 44.112.24.24 icmp_seq=5 ttl=252 time=1.686 ms

VPC> ping 44.112.26.26

84 bytes from 44.112.26.26 icmp_seq=1 ttl=252 time=1.150 ms
84 bytes from 44.112.26.26 icmp_seq=2 ttl=252 time=0.897 ms
84 bytes from 44.112.26.26 icmp_seq=3 ttl=252 time=0.915 ms
84 bytes from 44.112.26.26 icmp_seq=4 ttl=252 time=1.245 ms
84 bytes from 44.112.26.26 icmp_seq=5 ttl=252 time=1.292 ms


R18#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 44.112.24.250:39766 10.112.0.2:39766 44.112.24.24:39766 44.112.24.24:39766
icmp 44.112.24.250:40022 10.112.0.2:40022 44.112.24.24:40022 44.112.24.24:40022
icmp 44.112.24.250:40278 10.112.0.2:40278 44.112.24.24:40278 44.112.24.24:40278
icmp 44.112.24.250:40534 10.112.0.2:40534 44.112.24.24:40534 44.112.24.24:40534
icmp 44.112.24.250:40790 10.112.0.2:40790 44.112.24.24:40790 44.112.24.24:40790
icmp 44.112.26.250:42070 10.112.0.2:42070 44.112.26.26:42070 44.112.26.26:42070
icmp 44.112.26.250:42326 10.112.0.2:42326 44.112.26.26:42326 44.112.26.26:42326
icmp 44.112.26.250:42582 10.112.0.2:42582 44.112.26.26:42582 44.112.26.26:42582
icmp 44.112.26.250:42838 10.112.0.2:42838 44.112.26.26:42838 44.112.26.26:42838
icmp 44.112.26.250:43094 10.112.0.2:43094 44.112.26.26:43094 44.112.26.26:43094
icmp 44.112.24.250:44374 10.112.1.2:44374 44.112.24.24:44374 44.112.24.24:44374
icmp 44.112.24.250:44630 10.112.1.2:44630 44.112.24.24:44630 44.112.24.24:44630
icmp 44.112.24.250:44886 10.112.1.2:44886 44.112.24.24:44886 44.112.24.24:44886
icmp 44.112.24.250:45142 10.112.1.2:45142 44.112.24.24:45142 44.112.24.24:45142
icmp 44.112.24.250:45398 10.112.1.2:45398 44.112.24.24:45398 44.112.24.24:45398
icmp 44.112.26.250:46166 10.112.1.2:46166 44.112.26.26:46166 44.112.26.26:46166
icmp 44.112.26.250:46422 10.112.1.2:46422 44.112.26.26:46422 44.112.26.26:46422
icmp 44.112.26.250:46678 10.112.1.2:46678 44.112.26.26:46678 44.112.26.26:46678
icmp 44.112.26.250:46934 10.112.1.2:46934 44.112.26.26:46934 44.112.26.26:46934
icmp 44.112.26.250:47190 10.112.1.2:47190 44.112.26.26:47190 44.112.26.26:47190
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
ip nat inside source route-map RM-R28-R25-STNAT interface Ethernet1/1 overload
ip nat inside source route-map RM-R28-R26-STNAT interface Ethernet1/0 overload
!
ip access-list standard R28-STNAT-IN
 permit 10.111.4.0 0.0.0.255
!
route-map RM-R28-R26-STNAT permit 10
 match ip address R28-STNAT-IN
 match interface Ethernet1/0
!
route-map RM-R28-R25-STNAT permit 10
 match ip address R28-STNAT-IN
 match interface Ethernet1/1
!
```

### NAT

```
VPC30> ping 44.114.25.25

84 bytes from 44.114.25.25 icmp_seq=1 ttl=253 time=0.981 ms
84 bytes from 44.114.25.25 icmp_seq=2 ttl=253 time=1.566 ms
84 bytes from 44.114.25.25 icmp_seq=3 ttl=253 time=1.190 ms
84 bytes from 44.114.25.25 icmp_seq=4 ttl=253 time=1.457 ms
84 bytes from 44.114.25.25 icmp_seq=5 ttl=253 time=1.358 ms

VPC30> ping 44.114.26.26

84 bytes from 44.114.26.26 icmp_seq=1 ttl=254 time=1.335 ms
84 bytes from 44.114.26.26 icmp_seq=2 ttl=254 time=1.273 ms
84 bytes from 44.114.26.26 icmp_seq=3 ttl=254 time=1.072 ms
84 bytes from 44.114.26.26 icmp_seq=4 ttl=254 time=1.225 ms
84 bytes from 44.114.26.26 icmp_seq=5 ttl=254 time=1.732 ms

VPC31> ping 44.114.25.25

84 bytes from 44.114.25.25 icmp_seq=1 ttl=254 time=0.681 ms
84 bytes from 44.114.25.25 icmp_seq=2 ttl=254 time=0.829 ms
84 bytes from 44.114.25.25 icmp_seq=3 ttl=254 time=1.011 ms
84 bytes from 44.114.25.25 icmp_seq=4 ttl=254 time=1.189 ms
84 bytes from 44.114.25.25 icmp_seq=5 ttl=254 time=0.981 ms

VPC31> ping 44.114.26.26

84 bytes from 44.114.26.26 icmp_seq=1 ttl=253 time=1.431 ms
84 bytes from 44.114.26.26 icmp_seq=2 ttl=253 time=1.226 ms
84 bytes from 44.114.26.26 icmp_seq=3 ttl=253 time=1.311 ms
84 bytes from 44.114.26.26 icmp_seq=4 ttl=253 time=2.030 ms
84 bytes from 44.114.26.26 icmp_seq=5 ttl=253 time=1.086 ms


R28#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 44.114.26.28:98   10.111.4.66:98     44.114.26.26:98    44.114.26.26:98
icmp 44.114.26.28:354  10.111.4.66:354    44.114.26.26:354   44.114.26.26:354
icmp 44.114.26.28:61537 10.111.4.66:61537 44.114.25.25:61537 44.114.25.25:61537
icmp 44.114.26.28:61793 10.111.4.66:61793 44.114.25.25:61793 44.114.25.25:61793
icmp 44.114.26.28:62049 10.111.4.66:62049 44.114.25.25:62049 44.114.25.25:62049
icmp 44.114.26.28:62305 10.111.4.66:62305 44.114.25.25:62305 44.114.25.25:62305
icmp 44.114.26.28:62561 10.111.4.66:62561 44.114.25.25:62561 44.114.25.25:62561
icmp 44.114.26.28:64865 10.111.4.66:64865 44.114.26.26:64865 44.114.26.26:64865
icmp 44.114.26.28:65121 10.111.4.66:65121 44.114.26.26:65121 44.114.26.26:65121
icmp 44.114.26.28:65377 10.111.4.66:65377 44.114.26.26:65377 44.114.26.26:65377
icmp 44.114.25.28:1634 10.111.4.131:1634  44.114.25.25:1634  44.114.25.25:1634
icmp 44.114.25.28:1890 10.111.4.131:1890  44.114.25.25:1890  44.114.25.25:1890
icmp 44.114.25.28:2146 10.111.4.131:2146  44.114.25.25:2146  44.114.25.25:2146
icmp 44.114.25.28:2402 10.111.4.131:2402  44.114.25.25:2402  44.114.25.25:2402
icmp 44.114.25.28:2658 10.111.4.131:2658  44.114.25.25:2658  44.114.25.25:2658
icmp 44.114.25.28:4194 10.111.4.131:4194  44.114.26.26:4194  44.114.26.26:4194
icmp 44.114.25.28:4450 10.111.4.131:4450  44.114.26.26:4450  44.114.26.26:4450
icmp 44.114.25.28:4706 10.111.4.131:4706  44.114.26.26:4706  44.114.26.26:4706
icmp 44.114.25.28:4962 10.111.4.131:4962  44.114.26.26:4962  44.114.26.26:4962
icmp 44.114.25.28:5218 10.111.4.131:5218  44.114.26.26:5218  44.114.26.26:5218
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