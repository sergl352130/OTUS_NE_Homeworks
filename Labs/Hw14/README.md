# Лабораторная работа 14
+ ## IPSec over DMVPN
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw12/Network_topology.png?raw=true)

## Цели:
+ ### Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
+ ### DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги


### Этапы:
+ #### Шаг 1: Настройка GRE поверх IPSec между офисами Москва и С.-Петербург
+ #### Шаг 2: Настройка DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги
+ #### Шаг 3: Для настройки IPSec использовать CA и сертификаты
+ #### Шаг 4: Все узлы в офисах в лабораторной работе должны иметь IP связность 


#### R14:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
interface Loopback18
 description GRE+DMVPN
 ip address 40.40.40.40 255.255.255.255
!
interface Tunnel18
 description GRE to R18
 ip address 10.111.2.129 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 40.40.40.40
 tunnel destination 80.80.80.80
!
interface Tunnel114
 description DMVPN to R27-R28
 ip address 10.111.2.194 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp registration timeout 60
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source 40.40.40.40
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.111.3.14
 bgp log-neighbor-changes
 network 10.111.3.19 mask 255.255.255.255
 network 14.14.14.14 mask 255.255.255.255
 network 40.40.40.40 mask 255.255.255.255
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
 neighbor 22.111.22.22 route-map R22-Kitorn-OUT out
!
ip route 10.111.4.0 255.255.255.0 10.111.2.208
ip route 10.111.5.0 255.255.255.0 10.111.2.207
ip route 10.112.0.0 255.255.252.0 10.111.2.130
ip route 14.14.14.14 255.255.255.255 Null0
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
interface Loopback18
 description GRE+DMVPN
 ip address 50.50.50.50 255.255.255.255
!
interface Tunnel18
 description GRE to R18
 ip address 10.111.2.133 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 50.50.50.50
 tunnel destination 80.80.80.80
!
interface Tunnel115
 description DMVPN to R27-R28
 ip address 10.111.2.225 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp registration timeout 60
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source 50.50.50.50
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.111.3.15
 bgp log-neighbor-changes
 network 15.15.15.15 mask 255.255.255.255
 network 50.50.50.50 mask 255.255.255.255
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
 neighbor 33.111.21.21 route-map R21-Lamas-IN in
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip route 10.111.4.0 255.255.255.0 10.111.2.228
ip route 10.111.5.0 255.255.255.0 10.111.2.227
ip route 10.112.0.0 255.255.252.0 10.111.2.134
ip route 15.15.15.15 255.255.255.255 Null0
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
interface Loopback18
 description GRE to R14-R15
 ip address 80.80.80.80 255.255.255.255
!
interface Tunnel14
 description GRE to R14
 ip address 10.111.2.130 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 80.80.80.80
 tunnel destination 40.40.40.40
!
interface Tunnel15
 description GRE to R15
 ip address 10.111.2.134 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 80.80.80.80
 tunnel destination 50.50.50.50
!
router bgp 2042
 bgp router-id 10.112.3.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 10.112.3.18 mask 255.255.255.255
 network 18.18.18.16 mask 255.255.255.248
 network 80.80.80.80 mask 255.255.255.255
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26
ip route 10.111.0.0 255.255.252.0 10.111.2.129
ip route 10.111.0.0 255.255.252.0 10.111.2.133
ip route 18.18.18.16 255.255.255.248 Null0
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 6 permit 18.18.18.16/29
ip prefix-list Triada-OUT seq 7 permit 80.80.80.80/32
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
```

#### R27:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R27
!
interface Loopback27
 description DMVPN
 ip address 27.27.27.27 255.255.255.255
!
interface Tunnel114
 description DMVPN to R14
 ip address 10.111.2.207 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 40.40.40.40
 ip nhrp map 10.111.2.194 40.40.40.40
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.194
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 27.27.27.27
 tunnel mode gre multipoint
!
interface Tunnel115
 description DMVPN to R15
 ip address 10.111.2.227 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 50.50.50.50
 ip nhrp map 10.111.2.225 50.50.50.50
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.225
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 27.27.27.27
 tunnel mode gre multipoint
!
ip route 0.0.0.0 0.0.0.0 44.115.25.25
ip route 10.111.0.0 255.255.252.0 10.111.2.194
ip route 10.111.0.0 255.255.252.0 10.111.2.225
ip route 10.111.4.0 255.255.255.0 10.111.2.194
ip route 10.111.4.0 255.255.255.0 10.111.2.225
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
interface Loopback28
 description DMVPN
 ip address 28.28.28.28 255.255.255.255
!
interface Tunnel114
 description DMVPN to R14
 ip address 10.111.2.208 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 40.40.40.40
 ip nhrp map 10.111.2.194 40.40.40.40
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.194
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 28.28.28.28
 tunnel mode gre multipoint
!
interface Tunnel115
 description DMVPN to R15
 ip address 10.111.2.228 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 50.50.50.50
 ip nhrp map 10.111.2.225 50.50.50.50
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.225
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 28.28.28.28
 tunnel mode gre multipoint
!
ip route 0.0.0.0 0.0.0.0 44.114.25.25
ip route 0.0.0.0 0.0.0.0 44.114.26.26
ip route 10.111.0.0 255.255.252.0 10.111.2.194
ip route 10.111.0.0 255.255.252.0 10.111.2.225
ip route 10.111.5.0 255.255.255.0 10.111.2.194
ip route 10.111.5.0 255.255.255.0 10.111.2.225
!
```

### Проверка GRE

```
R14#show interfaces tunnel 18
Tunnel18 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R18
  Internet address is 10.111.2.129/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 40.40.40.40, destination 80.80.80.80
  Tunnel protocol/transport GRE/IP

R15#show interfaces tunnel 18
Tunnel18 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R18
  Internet address is 10.111.2.133/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 50.50.50.50, destination 80.80.80.80
  Tunnel protocol/transport GRE/IP

R18#show interfaces tunnel 14
Tunnel14 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R14
  Internet address is 10.111.2.130/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 80.80.80.80, destination 40.40.40.40
  Tunnel protocol/transport GRE/IP

R18#show interfaces tunnel 15
Tunnel15 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R15
  Internet address is 10.111.2.134/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 80.80.80.80, destination 50.50.50.50
  Tunnel protocol/transport GRE/IP

VPC1> ping 10.112.0.2

84 bytes from 10.112.0.2 icmp_seq=1 ttl=58 time=2.767 ms
84 bytes from 10.112.0.2 icmp_seq=2 ttl=58 time=2.749 ms
84 bytes from 10.112.0.2 icmp_seq=3 ttl=58 time=2.575 ms
84 bytes from 10.112.0.2 icmp_seq=4 ttl=58 time=2.920 ms
84 bytes from 10.112.0.2 icmp_seq=5 ttl=58 time=2.930 ms

VPC1> ping 10.112.1.2

84 bytes from 10.112.1.2 icmp_seq=1 ttl=58 time=2.561 ms
84 bytes from 10.112.1.2 icmp_seq=2 ttl=58 time=2.829 ms
84 bytes from 10.112.1.2 icmp_seq=3 ttl=58 time=2.302 ms
84 bytes from 10.112.1.2 icmp_seq=4 ttl=58 time=4.464 ms
84 bytes from 10.112.1.2 icmp_seq=5 ttl=58 time=2.231 ms

VPC7> ping 10.112.0.2

84 bytes from 10.112.0.2 icmp_seq=1 ttl=58 time=4.005 ms
84 bytes from 10.112.0.2 icmp_seq=2 ttl=58 time=2.347 ms
84 bytes from 10.112.0.2 icmp_seq=3 ttl=58 time=3.562 ms
84 bytes from 10.112.0.2 icmp_seq=4 ttl=58 time=2.715 ms
84 bytes from 10.112.0.2 icmp_seq=5 ttl=58 time=2.582 ms

VPC7> ping 10.112.1.2

84 bytes from 10.112.1.2 icmp_seq=1 ttl=58 time=4.192 ms
84 bytes from 10.112.1.2 icmp_seq=2 ttl=58 time=3.015 ms
84 bytes from 10.112.1.2 icmp_seq=3 ttl=58 time=2.758 ms
84 bytes from 10.112.1.2 icmp_seq=4 ttl=58 time=2.994 ms
84 bytes from 10.112.1.2 icmp_seq=5 ttl=58 time=2.551 ms

```

### Проверка DMVPN

```
R14#show interfaces tunnel 114
Tunnel114 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R27-R28
  Internet address is 10.111.2.194/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 40.40.40.40
  Tunnel protocol/transport multi-GRE/IP

R14#
R14#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel114, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 27.27.27.27        10.111.2.207    UP 00:54:58     D
     1 28.28.28.28        10.111.2.208    UP 00:54:27     D

R14#
R14#sh ip nhrp
10.111.2.207/32 via 10.111.2.207
   Tunnel114 created 00:55:09, expire 00:02:24
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 27.27.27.27 
10.111.2.208/32 via 10.111.2.208
   Tunnel114 created 00:54:37, expire 00:02:30
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 28.28.28.28 


R15#show interfaces tunnel 115
Tunnel115 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R27-R28
  Internet address is 10.111.2.225/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 50.50.50.50
  Tunnel protocol/transport multi-GRE/IP

R15#
R15#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel115, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 27.27.27.27        10.111.2.227    UP 00:55:27     D
     1 28.28.28.28        10.111.2.228    UP 00:55:48     D

R15#
R15#sh ip nhrp
10.111.2.227/32 via 10.111.2.227
   Tunnel115 created 00:55:35, expire 00:02:49
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 27.27.27.27 
10.111.2.228/32 via 10.111.2.228
   Tunnel115 created 00:55:56, expire 00:02:15
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 28.28.28.28


R27#show interfaces tunnel 114
Tunnel114 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R14
  Internet address is 10.111.2.207/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 27.27.27.27
  Tunnel protocol/transport multi-GRE/IP

R27#
R27#show interfaces tunnel 115
Tunnel115 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R15
  Internet address is 10.111.2.227/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 27.27.27.27
  Tunnel protocol/transport multi-GRE/IP

R27#
R27#show dmvpn       
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel114, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 40.40.40.40        10.111.2.194  NHRP 03:32:03     S
     0 UNKNOWN            10.111.2.208  NHRP    never    IX

Interface: Tunnel115, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 50.50.50.50        10.111.2.225    UP 03:28:17     S
     1 28.28.28.28        10.111.2.228    UP 00:00:30     D

R27#
R27#show ip nhrp
10.111.2.194/32 via 10.111.2.194
   Tunnel114 created 01:08:38, never expire 
   Type: static, Flags: used 
   NBMA address: 40.40.40.40 
10.111.2.225/32 via 10.111.2.225
   Tunnel115 created 01:08:38, never expire 
   Type: static, Flags: used 
   NBMA address: 50.50.50.50 


R28#show interfaces tunnel 114
Tunnel114 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R14
  Internet address is 10.111.2.208/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 28.28.28.28
  Tunnel protocol/transport multi-GRE/IP

R28#
R28#show interfaces tunnel 115
Tunnel115 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R15
  Internet address is 10.111.2.228/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 28.28.28.28
  Tunnel protocol/transport multi-GRE/IP

R28#
R28#show dmvpn       
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel114, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 40.40.40.40        10.111.2.194  NHRP 02:24:26     S
     0 UNKNOWN            10.111.2.207  NHRP    never    IX

Interface: Tunnel115, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 50.50.50.50        10.111.2.225    UP 02:25:15     S
     1 27.27.27.27        10.111.2.227    UP 00:00:37     D


R28#
R28#show ip nhrp
10.111.2.194/32 via 10.111.2.194
   Tunnel114 created 02:10:44, never expire 
   Type: static, Flags: 
   NBMA address: 40.40.40.40 
10.111.2.225/32 via 10.111.2.225
   Tunnel115 created 02:09:18, never expire 
   Type: static, Flags: 
   NBMA address: 50.50.50.50


VPC1> ping 10.111.4.65 

84 bytes from 10.111.4.65 icmp_seq=1 ttl=252 time=2.452 ms
84 bytes from 10.111.4.65 icmp_seq=2 ttl=252 time=2.263 ms
84 bytes from 10.111.4.65 icmp_seq=3 ttl=252 time=2.658 ms
84 bytes from 10.111.4.65 icmp_seq=4 ttl=252 time=2.251 ms
84 bytes from 10.111.4.65 icmp_seq=5 ttl=252 time=1.989 ms

VPC1> ping 10.111.4.129

84 bytes from 10.111.4.129 icmp_seq=1 ttl=252 time=2.574 ms
84 bytes from 10.111.4.129 icmp_seq=2 ttl=252 time=2.709 ms
84 bytes from 10.111.4.129 icmp_seq=3 ttl=252 time=2.896 ms
84 bytes from 10.111.4.129 icmp_seq=4 ttl=252 time=2.622 ms
84 bytes from 10.111.4.129 icmp_seq=5 ttl=252 time=2.671 ms

VPC1> ping 10.111.5.129

84 bytes from 10.111.5.129 icmp_seq=1 ttl=252 time=3.186 ms
84 bytes from 10.111.5.129 icmp_seq=2 ttl=252 time=3.304 ms
84 bytes from 10.111.5.129 icmp_seq=3 ttl=252 time=2.546 ms
84 bytes from 10.111.5.129 icmp_seq=4 ttl=252 time=2.521 ms
84 bytes from 10.111.5.129 icmp_seq=5 ttl=252 time=8.531 ms


VPC7> ping 10.111.4.65

84 bytes from 10.111.4.65 icmp_seq=1 ttl=252 time=2.718 ms
84 bytes from 10.111.4.65 icmp_seq=2 ttl=252 time=2.804 ms
84 bytes from 10.111.4.65 icmp_seq=3 ttl=252 time=2.426 ms
84 bytes from 10.111.4.65 icmp_seq=4 ttl=252 time=2.278 ms
84 bytes from 10.111.4.65 icmp_seq=5 ttl=252 time=2.668 ms

VPC7> ping 10.111.4.129

84 bytes from 10.111.4.129 icmp_seq=1 ttl=252 time=1.996 ms
84 bytes from 10.111.4.129 icmp_seq=2 ttl=252 time=2.292 ms
84 bytes from 10.111.4.129 icmp_seq=3 ttl=252 time=2.090 ms
84 bytes from 10.111.4.129 icmp_seq=4 ttl=252 time=3.190 ms
84 bytes from 10.111.4.129 icmp_seq=5 ttl=252 time=3.112 ms

VPC7> ping 10.111.5.129

84 bytes from 10.111.5.129 icmp_seq=1 ttl=252 time=2.980 ms
84 bytes from 10.111.5.129 icmp_seq=2 ttl=252 time=2.995 ms
84 bytes from 10.111.5.129 icmp_seq=3 ttl=252 time=3.127 ms
84 bytes from 10.111.5.129 icmp_seq=4 ttl=252 time=2.405 ms
84 bytes from 10.111.5.129 icmp_seq=5 ttl=252 time=2.908 ms

```