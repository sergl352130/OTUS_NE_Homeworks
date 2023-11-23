# Лабораторная работа 10
+ ## iBGP
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw09/Network_topology_BGP1.png?raw=true)

## Цели:
+ ### Настроить iBGP в офисе Москва
+ ### Настроить iBGP в сети провайдера Триада
+ ### Организовать полную IP связность всех сетей

### Этапы:
+ #### Шаг 1: Настройка iBGP в офисе Москва между маршрутизаторами R14 и R15
+ #### Шаг 2: Настройка iBGP в провайдере Триада, с использованием RR
+ #### Шаг 3: Настройка офиса Москва так, чтобы приоритетным провайдером стал Ламас
+ #### Шаг 4: Настройка офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно

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
 network 22.111.22.0 mask 255.255.255.0
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
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
 network 33.111.21.0 mask 255.255.255.0
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
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
 network 44.112.24.0 mask 255.255.255.0
 network 44.112.26.0 mask 255.255.255.0
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.26.26 remote-as 520
 maximum-paths 2
!
```

```
R18#show ip bgp summary
BGP router identifier 10.112.3.18, local AS number 2042
BGP table version is 25, main routing table version 25
9 network entries using 1260 bytes of memory
12 path entries using 960 bytes of memory
3 multipath network entries and 6 multipath paths
6/6 BGP path/bestpath attribute entries using 864 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3180 total bytes of memory
BGP activity 10/1 prefixes, 20/8 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
44.112.24.24    4          520     187     188       25    0    0 02:42:34        7
44.112.26.26    4          520     188     188       25    0    0 02:42:33        3
R18#
R18#show ip bgp
BGP table version is 25, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.112.24.24                           0 520 301 101 i
 *>  10.33.33.21/32   44.112.24.24                           0 520 301 i
 *>  22.111.22.0/24   44.112.24.24                           0 520 301 1001 i
 *>  33.111.21.0/24   44.112.24.24                           0 520 301 1001 i
 *>  44.112.24.0/24   0.0.0.0                  0         32768 i
 *>  44.112.26.0/24   0.0.0.0                  0         32768 i
 *m  44.114.25.0/24   44.112.24.24                           0 520 i
 *>                   44.112.26.26                           0 520 i
 *m  44.114.26.0/24   44.112.24.24                           0 520 i
 *>                   44.112.26.26             0             0 520 i
 *m  44.115.25.0/24   44.112.24.24                           0 520 i
 *>                   44.112.26.26                           0 520 i
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
 neighbor 44.33.24.24 remote-as 520
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
 neighbor 33.22.21.21 remote-as 301
 neighbor 44.22.23.23 remote-as 520
!
```

```

```

#### R23:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R23
!
interface Loopback0
 description R23 Mgmt
 ip address 10.44.44.23 255.255.255.255
 ip router isis 44
!         
interface Ethernet0/0
 description to R22 Kitorn
 ip address 44.22.23.23 255.255.255.0
!
interface Ethernet0/1
 description p2p to R25
 ip address 10.44.44.1 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R24
 ip address 10.44.44.5 255.255.255.252
 ip router isis 44
!
router bgp 520
 bgp router-id 10.44.44.23
 bgp log-neighbor-changes
 neighbor 10.44.44.25 remote-as 520
 neighbor 44.22.23.22 remote-as 101
!
```

```
R23#show ip bgp summary
BGP router identifier 10.44.44.23, local AS number 520
BGP table version is 16, main routing table version 16
9 network entries using 1260 bytes of memory
9 path entries using 720 bytes of memory
5/5 BGP path/bestpath attribute entries using 720 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2844 total bytes of memory
BGP activity 10/1 prefixes, 14/5 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.25     4          520      72      67       16    0    0 00:53:40        5
44.22.23.22     4          101     148     146       16    0    0 02:03:59        4

R23#
R23#show ip bgp
BGP table version is 16, local router ID is 10.44.44.23
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.22.23.22              0             0 101 i
 *>  10.33.33.21/32   44.22.23.22                            0 101 301 i
 *>  22.111.22.0/24   44.22.23.22                            0 101 1001 i
 *>  33.111.21.0/24   44.22.23.22                            0 101 1001 i
 *>i 44.112.24.0/24   44.112.24.18             0    100      0 2042 i
 * i 44.112.26.0/24   44.112.26.18             0    100      0 2042 i
 *>i 44.114.25.0/24   10.44.44.25              0    100      0 i
 *>i 44.114.26.0/24   10.44.44.26              0    100      0 i
 *>i 44.115.25.0/24   10.44.44.25              0    100      0 i
```

#### R24:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R24
!
interface Loopback0
 description R24 Mgmt
 ip address 10.44.44.24 255.255.255.255
 ip router isis 44
!
interface Ethernet0/0
 description to R21 Lamas
 ip address 44.33.24.24 255.255.255.0
!
interface Ethernet0/1
 description p2p to R26
 ip address 10.44.44.9 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R23
 ip address 10.44.44.6 255.255.255.252
 ip router isis 44
!
interface Ethernet1/0
 description to R18 SPb
 ip address 44.112.24.24 255.255.255.0
!
router bgp 520
 bgp router-id 10.44.44.24
 bgp log-neighbor-changes
 neighbor 10.44.44.25 remote-as 520
 neighbor 44.33.24.21 remote-as 301
 neighbor 44.112.24.18 remote-as 2042
!
```

```
R24#show ip bgp summary
BGP router identifier 10.44.44.24, local AS number 520
BGP table version is 13, main routing table version 13
9 network entries using 1260 bytes of memory
10 path entries using 800 bytes of memory
6/5 BGP path/bestpath attribute entries using 864 bytes of memory
1 BGP rrinfo entries using 24 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3044 total bytes of memory
BGP activity 10/1 prefixes, 12/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.25     4          520      75      72       13    0    0 00:55:55        4
44.33.24.21     4          301     150     148       13    0    0 02:06:35        4
44.112.24.18    4         2042     148     148       13    0    0 02:06:38        2

R24#
R24#show ip bgp
BGP table version is 13, local router ID is 10.44.44.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.33.24.21                            0 301 101 i
 *>  10.33.33.21/32   44.33.24.21              0             0 301 i
 *>  22.111.22.0/24   44.33.24.21                            0 301 1001 i
 *>  33.111.21.0/24   44.33.24.21                            0 301 1001 i
 r>  44.112.24.0/24   44.112.24.18             0             0 2042 i
 * i 44.112.26.0/24   44.112.26.18             0    100      0 2042 i
 *>                   44.112.24.18             0             0 2042 i
 *>i 44.114.25.0/24   10.44.44.25              0    100      0 i
 *>i 44.114.26.0/24   10.44.44.26              0    100      0 i
 *>i 44.115.25.0/24   10.44.44.25              0    100      0 i
```

#### R25:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R25
!
interface Loopback0
 description R25 Mgmt
 ip address 10.44.44.25 255.255.255.255
 ip router isis 44
!
interface Ethernet0/1
 description p2p to R23
 ip address 10.44.44.2 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R26
 ip address 10.44.44.13 255.255.255.252
 ip router isis 44
!
interface Ethernet1/0
 description to R27 Lab-gi
 ip address 44.115.25.25 255.255.255.0
!
interface Ethernet1/1
 description to R28 Chok
 ip address 44.114.25.25 255.255.255.0
!
router bgp 520
 bgp log-neighbor-changes
 network 44.114.25.0 mask 255.255.255.0
 network 44.115.25.0 mask 255.255.255.0
 neighbor RR-Triada peer-group
 neighbor RR-Triada remote-as 520
 neighbor RR-Triada update-source Loopback0
 neighbor RR-Triada route-reflector-client
 neighbor 10.44.44.23 peer-group RR-Triada
 neighbor 10.44.44.24 peer-group RR-Triada
 neighbor 10.44.44.26 peer-group RR-Triada
!
```

```
R25#show ip bgp summary
BGP router identifier 10.44.44.25, local AS number 520
BGP table version is 18, main routing table version 18
9 network entries using 1260 bytes of memory
15 path entries using 1200 bytes of memory
9/3 BGP path/bestpath attribute entries using 1296 bytes of memory
7 BGP AS-PATH entries using 168 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3924 total bytes of memory
BGP activity 10/1 prefixes, 18/3 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.23     4          520      70      77       18    0    0 00:57:20        4
10.44.44.24     4          520      73      76       18    0    0 00:57:00        6
10.44.44.26     4          520      69      77       18    0    0 00:56:41        3

R25#
R25#show ip bgp
BGP table version is 18, local router ID is 10.44.44.25
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.22.22.22/32   44.33.24.21              0    100      0 301 101 i
 * i                  44.22.23.22              0    100      0 101 i
 * i 10.33.33.21/32   44.22.23.22              0    100      0 101 301 i
 * i                  44.33.24.21              0    100      0 301 i
 * i 22.111.22.0/24   44.33.24.21              0    100      0 301 1001 i
 * i                  44.22.23.22              0    100      0 101 1001 i
 * i 33.111.21.0/24   44.33.24.21              0    100      0 301 1001 i
 * i                  44.22.23.22              0    100      0 101 1001 i
 * i 44.112.24.0/24   44.112.26.18             0    100      0 2042 i
 *>i                  44.112.24.18             0    100      0 2042 i
 *>i 44.112.26.0/24   44.112.26.18             0    100      0 2042 i
 * i                  44.112.24.18             0    100      0 2042 i
 *>  44.114.25.0/24   0.0.0.0                  0         32768 i
 *>i 44.114.26.0/24   10.44.44.26              0    100      0 i
 *>  44.115.25.0/24   0.0.0.0                  0         32768 i
```

#### R26:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R26
!
interface Loopback0
 description R26 Mgmt
 ip address 10.44.44.26 255.255.255.255
 ip router isis 44
!         
interface Ethernet0/1
 description p2p to R24
 ip address 10.44.44.10 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R25
 ip address 10.44.44.14 255.255.255.252
 ip router isis 44
!
interface Ethernet1/0
 description to R28 Chok
 ip address 44.114.26.26 255.255.255.0
!
interface Ethernet1/1
 description to R18 SPb
 ip address 44.112.26.26 255.255.255.0
!
router bgp 520
 bgp router-id 10.44.44.26
 bgp log-neighbor-changes
 network 44.114.26.0 mask 255.255.255.0
 neighbor 10.44.44.25 remote-as 520
 neighbor 44.112.26.18 remote-as 2042
!
```

```
R26#show ip bgp summary
BGP router identifier 10.44.44.26, local AS number 520
BGP table version is 18, main routing table version 18
5 network entries using 700 bytes of memory
6 path entries using 480 bytes of memory
4/3 BGP path/bestpath attribute entries using 576 bytes of memory
1 BGP rrinfo entries using 24 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1804 total bytes of memory
BGP activity 10/5 prefixes, 12/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.25     4          520      78      71       18    0    0 00:58:18        3
44.112.26.18    4         2042     151     152       18    0    0 02:09:19        2

R26#
R26#show ip bgp
BGP table version is 18, local router ID is 10.44.44.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 44.112.24.0/24   44.112.24.18             0    100      0 2042 i
 *>                   44.112.26.18             0             0 2042 i
 r>  44.112.26.0/24   44.112.26.18             0             0 2042 i
 *>i 44.114.25.0/24   10.44.44.25              0    100      0 i
 *>  44.114.26.0/24   0.0.0.0                  0         32768 i
 *>i 44.115.25.0/24   10.44.44.25              0    100      0 i
```

+ #### Шаг 5: Все сети в лабораторной работе должны иметь IP связность

```
R14#ping 44.112.24.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 44.112.24.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

```
R18#ping 33.111.21.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 33.111.21.15, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```