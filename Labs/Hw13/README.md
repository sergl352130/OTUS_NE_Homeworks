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
 ip tcp adjust-mss 1360
 tunnel source 40.40.40.40
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.111.3.14
 bgp log-neighbor-changes
 network 10.111.3.14 mask 255.255.255.255
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
 ip tcp adjust-mss 1360
 tunnel source 50.50.50.50
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.111.3.15
 bgp log-neighbor-changes
 network 10.111.3.15 mask 255.255.255.255
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
 ip mtu 1400
 ip nhrp map multicast 40.40.40.40
 ip nhrp map 10.111.2.194 40.40.40.40
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.194
 ip nhrp registration timeout 60
 ip tcp adjust-mss 1360
 tunnel source 27.27.27.27
 tunnel destination 40.40.40.40
!
interface Tunnel115
 description DMVPN to R15
 ip address 10.111.2.227 255.255.255.224
 ip mtu 1400
 ip nhrp map multicast 50.50.50.50
 ip nhrp map 10.111.2.225 50.50.50.50
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.225
 ip nhrp registration timeout 60
 ip tcp adjust-mss 1360
 tunnel source 27.27.27.27
 tunnel destination 50.50.50.50
!
ip route 0.0.0.0 0.0.0.0 44.115.25.25
ip route 10.111.0.0 255.255.252.0 10.111.2.194
ip route 10.111.0.0 255.255.252.0 10.111.2.225
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
 ip mtu 1400
 ip nhrp map multicast 40.40.40.40
 ip nhrp map 10.111.2.194 40.40.40.40
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.194
 ip nhrp registration timeout 60
 ip tcp adjust-mss 1360
 tunnel source 28.28.28.28
 tunnel destination 40.40.40.40
!
interface Tunnel115
 description DMVPN to R15
 ip address 10.111.2.228 255.255.255.224
 ip mtu 1400
 ip nhrp map multicast 50.50.50.50
 ip nhrp map 10.111.2.225 50.50.50.50
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.225
 ip nhrp registration timeout 60
 ip tcp adjust-mss 1360
 tunnel source 28.28.28.28
 tunnel destination 50.50.50.50
!
ip route 0.0.0.0 0.0.0.0 44.114.25.25
ip route 0.0.0.0 0.0.0.0 44.114.26.26
ip route 10.111.0.0 255.255.252.0 10.111.2.194
ip route 10.111.0.0 255.255.252.0 10.111.2.225
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