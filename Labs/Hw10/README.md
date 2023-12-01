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
+ #### Шаг 5: Все сети в лабораторной работе должны иметь IP связность


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
 neighbor 22.111.22.22 route-map R22-Kitorn-PREP out
!
route-map R22-Kitorn-PREP permit 10
 set as-path prepend 1001 1001
!
```

```
R14#show ip bgp summary
BGP router identifier 10.111.3.14, local AS number 1001
BGP table version is 85, main routing table version 85
12 network entries using 1680 bytes of memory
22 path entries using 1760 bytes of memory
10/6 BGP path/bestpath attribute entries using 1440 bytes of memory
8 BGP AS-PATH entries using 208 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 5088 total bytes of memory
BGP activity 45/33 prefixes, 69/47 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.111.3.15     4         1001     112      83       85    0    0 00:38:14       11
22.111.22.22    4          101     102     117       85    0    0 00:37:53       10
R14#
R14#show ip bgp        
BGP table version is 85, local router ID is 10.111.3.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   10.22.22.22/32   22.111.22.22             0             0 101 i
 *>i                  10.111.3.15              0   1000      0 301 101 i
 *   10.33.33.21/32   22.111.22.22                           0 101 301 i
 *>i                  10.111.3.15              0   1000      0 301 i
 *>i 10.44.44.23/32   10.111.3.15              0   1000      0 301 520 i
 *                    22.111.22.22                           0 101 520 i
 *>i 10.44.44.24/32   10.111.3.15              0   1000      0 301 520 i
 *                    22.111.22.22                           0 101 520 i
 *>i 10.44.44.25/32   10.111.3.15              0   1000      0 301 520 i
 *                    22.111.22.22                           0 101 520 i
 *   10.44.44.26/32   22.111.22.22                           0 101 520 i
 *>i                  10.111.3.15              0   1000      0 301 520 i
 *>  10.111.3.14/32   0.0.0.0                  0         32768 i
 r>i 10.111.3.15/32   10.111.3.15              0    100      0 i
     Network          Next Hop            Metric LocPrf Weight Path
 *   10.112.3.18/32   22.111.22.22                           0 101 301 520 2042 i
 *>i                  10.111.3.15              0   1000      0 301 520 2042 i
 *   44.114.25.0/24   22.111.22.22                           0 101 520 i
 *>i                  10.111.3.15              0   1000      0 301 520 i
 *   44.114.26.0/24   22.111.22.22                           0 101 520 i
 *>i                  10.111.3.15              0   1000      0 301 520 i
 *   44.115.25.0/24   22.111.22.22                           0 101 520 i
 *>i                  10.111.3.15              0   1000      0 301 520 i
R14#
R14#show ip route      
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 31 subnets, 4 masks
B        10.22.22.22/32 [200/0] via 10.111.3.15, 00:38:27
B        10.33.33.21/32 [200/0] via 10.111.3.15, 00:38:27
B        10.44.44.23/32 [200/0] via 10.111.3.15, 00:05:32
B        10.44.44.24/32 [200/0] via 10.111.3.15, 00:05:01
B        10.44.44.25/32 [200/0] via 10.111.3.15, 00:04:06
B        10.44.44.26/32 [200/0] via 10.111.3.15, 00:03:36
O IA     10.111.0.0/24 [110/21] via 10.111.2.9, 00:39:34, Ethernet0/0
O IA     10.111.1.0/24 [110/21] via 10.111.2.21, 00:39:34, Ethernet0/1
O IA     10.111.2.0/30 [110/20] via 10.111.2.9, 00:39:34, Ethernet0/0
O IA     10.111.2.4/30 [110/20] via 10.111.2.21, 00:39:34, Ethernet0/1
C        10.111.2.8/30 is directly connected, Ethernet0/0
L        10.111.2.10/32 is directly connected, Ethernet0/0
O        10.111.2.12/30 [110/20] via 10.111.2.26, 00:39:34, Ethernet0/2
                        [110/20] via 10.111.2.9, 00:39:34, Ethernet0/0
O        10.111.2.16/30 [110/20] via 10.111.2.26, 00:39:34, Ethernet0/2
                        [110/20] via 10.111.2.21, 00:39:34, Ethernet0/1
C        10.111.2.20/30 is directly connected, Ethernet0/1
L        10.111.2.22/32 is directly connected, Ethernet0/1
C        10.111.2.24/30 is directly connected, Ethernet0/2
L        10.111.2.25/32 is directly connected, Ethernet0/2
C        10.111.2.28/30 is directly connected, Ethernet0/3
L        10.111.2.29/32 is directly connected, Ethernet0/3
O IA     10.111.2.32/30 [110/20] via 10.111.2.26, 00:39:34, Ethernet0/2
O IA     10.111.3.4/32 [110/21] via 10.111.2.9, 00:39:34, Ethernet0/0
O IA     10.111.3.5/32 [110/21] via 10.111.2.21, 00:39:34, Ethernet0/1
O        10.111.3.12/32 [110/11] via 10.111.2.9, 00:39:34, Ethernet0/0
O        10.111.3.13/32 [110/11] via 10.111.2.21, 00:39:34, Ethernet0/1
C        10.111.3.14/32 is directly connected, Loopback0
O        10.111.3.15/32 [110/11] via 10.111.2.26, 00:39:34, Ethernet0/2
O        10.111.3.19/32 [110/11] via 10.111.2.30, 00:39:34, Ethernet0/3
O IA     10.111.3.20/32 [110/21] via 10.111.2.26, 00:39:34, Ethernet0/2
O IA     10.111.3.96/28 [110/21] via 10.111.2.21, 00:39:34, Ethernet0/1
                        [110/21] via 10.111.2.9, 00:39:34, Ethernet0/0
B        10.112.3.18/32 [200/0] via 10.111.3.15, 00:33:18
      22.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        22.111.22.0/24 is directly connected, Ethernet1/0
L        22.111.22.14/32 is directly connected, Ethernet1/0
      44.0.0.0/24 is subnetted, 3 subnets
B        44.114.25.0 [200/0] via 10.111.3.15, 00:38:27
B        44.114.26.0 [200/0] via 10.111.3.15, 00:38:27
B        44.115.25.0 [200/0] via 10.111.3.15, 00:38:27
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
!
route-map R21-Lamas-IN permit 10
 set local-preference 1000
!
```

```
R15#show ip bgp summary
BGP router identifier 10.111.3.15, local AS number 1001
BGP table version is 93, main routing table version 93
12 network entries using 1680 bytes of memory
12 path entries using 960 bytes of memory
10/6 BGP path/bestpath attribute entries using 1440 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4176 total bytes of memory
BGP activity 45/33 prefixes, 55/43 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.111.3.14     4         1001      81     110       93    0    0 00:36:52        1
33.111.21.21    4          301     654     636       93    0    0 09:13:58       10
R15#
R15#show ip bgp
BGP table version is 93, local router ID is 10.111.3.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   33.111.21.21                 1000      0 301 101 i
 *>  10.33.33.21/32   33.111.21.21             0   1000      0 301 i
 *>  10.44.44.23/32   33.111.21.21                 1000      0 301 520 i
 *>  10.44.44.24/32   33.111.21.21                 1000      0 301 520 i
 *>  10.44.44.25/32   33.111.21.21                 1000      0 301 520 i
 *>  10.44.44.26/32   33.111.21.21                 1000      0 301 520 i
 r>i 10.111.3.14/32   10.111.3.14              0    100      0 i
 *>  10.111.3.15/32   0.0.0.0                  0         32768 i
 *>  10.112.3.18/32   33.111.21.21                 1000      0 301 520 2042 i
 *>  44.114.25.0/24   33.111.21.21                 1000      0 301 520 i
 *>  44.114.26.0/24   33.111.21.21                 1000      0 301 520 i
 *>  44.115.25.0/24   33.111.21.21                 1000      0 301 520 i
R15#
R15#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 31 subnets, 4 masks
B        10.22.22.22/32 [20/0] via 33.111.21.21, 09:12:55
B        10.33.33.21/32 [20/0] via 33.111.21.21, 09:13:15
B        10.44.44.23/32 [20/0] via 33.111.21.21, 00:04:17
B        10.44.44.24/32 [20/0] via 33.111.21.21, 00:03:46
B        10.44.44.25/32 [20/0] via 33.111.21.21, 00:02:51
B        10.44.44.26/32 [20/0] via 33.111.21.21, 00:02:21
O IA     10.111.0.0/24 [110/21] via 10.111.2.13, 09:13:47, Ethernet0/1
O IA     10.111.1.0/24 [110/21] via 10.111.2.17, 09:13:47, Ethernet0/0
O IA     10.111.2.0/30 [110/20] via 10.111.2.13, 09:14:14, Ethernet0/1
O IA     10.111.2.4/30 [110/20] via 10.111.2.17, 09:14:14, Ethernet0/0
O        10.111.2.8/30 [110/20] via 10.111.2.25, 09:14:14, Ethernet0/2
                       [110/20] via 10.111.2.13, 09:14:14, Ethernet0/1
C        10.111.2.12/30 is directly connected, Ethernet0/1
L        10.111.2.14/32 is directly connected, Ethernet0/1
C        10.111.2.16/30 is directly connected, Ethernet0/0
L        10.111.2.18/32 is directly connected, Ethernet0/0
O        10.111.2.20/30 [110/20] via 10.111.2.25, 09:14:14, Ethernet0/2
                        [110/20] via 10.111.2.17, 09:14:14, Ethernet0/0
C        10.111.2.24/30 is directly connected, Ethernet0/2
L        10.111.2.26/32 is directly connected, Ethernet0/2
O IA     10.111.2.28/30 [110/20] via 10.111.2.25, 09:14:14, Ethernet0/2
C        10.111.2.32/30 is directly connected, Ethernet0/3
L        10.111.2.33/32 is directly connected, Ethernet0/3
O IA     10.111.3.4/32 [110/21] via 10.111.2.13, 09:14:13, Ethernet0/1
O IA     10.111.3.5/32 [110/21] via 10.111.2.17, 09:14:13, Ethernet0/0
O        10.111.3.12/32 [110/11] via 10.111.2.13, 09:14:14, Ethernet0/1
O        10.111.3.13/32 [110/11] via 10.111.2.17, 09:14:14, Ethernet0/0
O        10.111.3.14/32 [110/11] via 10.111.2.25, 09:14:14, Ethernet0/2
C        10.111.3.15/32 is directly connected, Loopback0
O IA     10.111.3.19/32 [110/21] via 10.111.2.25, 09:14:14, Ethernet0/2
O        10.111.3.20/32 [110/11] via 10.111.2.34, 09:14:14, Ethernet0/3
O IA     10.111.3.96/28 [110/21] via 10.111.2.17, 09:13:47, Ethernet0/0
                        [110/21] via 10.111.2.13, 09:14:13, Ethernet0/1
B        10.112.3.18/32 [20/0] via 33.111.21.21, 00:32:03
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        33.111.21.0/24 is directly connected, Ethernet1/0
L        33.111.21.15/32 is directly connected, Ethernet1/0
      44.0.0.0/24 is subnetted, 3 subnets
B        44.114.25.0 [20/0] via 33.111.21.21, 09:12:55
B        44.114.26.0 [20/0] via 33.111.21.21, 09:12:55
B        44.115.25.0 [20/0] via 33.111.21.21, 09:12:55
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
 neighbor 44.112.26.26 remote-as 520
 maximum-paths 2
!
```

```
R18#show ip bgp summary
BGP router identifier 10.112.3.18, local AS number 2042
BGP table version is 109, main routing table version 109
12 network entries using 1680 bytes of memory
19 path entries using 1520 bytes of memory
7 multipath network entries and 14 multipath paths
6/6 BGP path/bestpath attribute entries using 864 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4160 total bytes of memory
BGP activity 46/34 prefixes, 53/34 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
44.112.24.24    4          520     662     665      109    0    0 09:14:01       11
44.112.26.26    4          520     631     665      109    0    0 09:14:01        7
R18#
R18#show ip bgp
BGP table version is 109, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.112.24.24                           0 520 301 101 i
 *>  10.33.33.21/32   44.112.24.24                           0 520 301 i
 *m  10.44.44.23/32   44.112.24.24                           0 520 i
 *>                   44.112.26.26                           0 520 i
 *m  10.44.44.24/32   44.112.26.26                           0 520 i
 *>                   44.112.24.24             0             0 520 i
 *m  10.44.44.25/32   44.112.24.24                           0 520 i
 *>                   44.112.26.26                           0 520 i
 *m  10.44.44.26/32   44.112.26.26             0             0 520 i
 *>                   44.112.24.24                           0 520 i
 *>  10.111.3.14/32   44.112.24.24                           0 520 301 1001 i
 *>  10.111.3.15/32   44.112.24.24                           0 520 301 1001 i
 *>  10.112.3.18/32   0.0.0.0                  0         32768 i
 *m  44.114.25.0/24   44.112.24.24                           0 520 i
 *>                   44.112.26.26                           0 520 i
          
R18# 
R18#show ip route      
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 44.112.24.24 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 44.112.24.24
      10.0.0.0/8 is variably subnetted, 21 subnets, 3 masks
B        10.22.22.22/32 [20/0] via 44.112.24.24, 09:13:25
B        10.33.33.21/32 [20/0] via 44.112.24.24, 09:13:56
B        10.44.44.23/32 [20/0] via 44.112.26.26, 00:05:19
                        [20/0] via 44.112.24.24, 00:05:19
B        10.44.44.24/32 [20/0] via 44.112.26.26, 00:04:19
                        [20/0] via 44.112.24.24, 00:04:19
B        10.44.44.25/32 [20/0] via 44.112.26.26, 00:03:23
                        [20/0] via 44.112.24.24, 00:03:23
B        10.44.44.26/32 [20/0] via 44.112.26.26, 00:02:53
                        [20/0] via 44.112.24.24, 00:02:53
B        10.111.3.14/32 [20/0] via 44.112.24.24, 00:35:50
B        10.111.3.15/32 [20/0] via 44.112.24.24, 00:34:49
D        10.112.0.0/24 [90/1541120] via 10.112.2.17, 09:14:27, Ethernet0/0
D        10.112.1.0/24 [90/1541120] via 10.112.2.21, 09:14:52, Ethernet0/1
D        10.112.2.0/24 [90/1536000] via 10.112.2.21, 09:14:59, Ethernet0/1
                       [90/1536000] via 10.112.2.17, 09:14:59, Ethernet0/0
C        10.112.2.16/30 is directly connected, Ethernet0/0
L        10.112.2.18/32 is directly connected, Ethernet0/0
C        10.112.2.20/30 is directly connected, Ethernet0/1
L        10.112.2.22/32 is directly connected, Ethernet0/1
D        10.112.3.9/32 [90/1536640] via 10.112.2.17, 09:14:54, Ethernet0/0
D        10.112.3.10/32 [90/1536640] via 10.112.2.21, 09:14:54, Ethernet0/1
D        10.112.3.16/32 [90/1024640] via 10.112.2.21, 09:14:54, Ethernet0/1
D        10.112.3.17/32 [90/1024640] via 10.112.2.17, 09:14:54, Ethernet0/0
C        10.112.3.18/32 is directly connected, Loopback0
D        10.112.3.32/32 [90/1536640] via 10.112.2.21, 09:14:54, Ethernet0/1
      44.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
C        44.112.24.0/24 is directly connected, Ethernet1/0
L        44.112.24.18/32 is directly connected, Ethernet1/0
C        44.112.26.0/24 is directly connected, Ethernet1/1
L        44.112.26.18/32 is directly connected, Ethernet1/1
B        44.114.25.0/24 [20/0] via 44.112.26.26, 09:13:56
                        [20/0] via 44.112.24.24, 09:13:56
B        44.114.26.0/24 [20/0] via 44.112.26.26, 09:13:56
                        [20/0] via 44.112.24.24, 09:13:56
B        44.115.25.0/24 [20/0] via 44.112.26.26, 09:13:56
                        [20/0] via 44.112.24.24, 09:13:56
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
R21#show ip bgp summary
BGP router identifier 10.33.33.21, local AS number 301
BGP table version is 14, main routing table version 14
12 network entries using 1680 bytes of memory
21 path entries using 1680 bytes of memory
10/7 BGP path/bestpath attribute entries using 1440 bytes of memory
7 BGP AS-PATH entries using 168 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4968 total bytes of memory
BGP activity 12/0 prefixes, 22/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
33.22.21.22     4          101      92      94       14    0    0 01:16:34        9
33.111.21.15    4         1001      88      94       14    0    0 01:16:35        2
44.33.24.24     4          520      93      94       14    0    0 01:16:37        9
R21#
R21#show ip bgp
BGP table version is 14, local router ID is 10.33.33.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   10.22.22.22/32   44.33.24.24                            0 520 101 i
 *>                   33.22.21.22              0             0 101 i
 *>  10.33.33.21/32   0.0.0.0                  0         32768 i
 *>  10.44.44.23/32   44.33.24.24                            0 520 i
 *                    33.22.21.22                            0 101 520 i
 *   10.44.44.24/32   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24              0             0 520 i
 *   10.44.44.25/32   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
 *   10.44.44.26/32   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
 *>  10.111.3.14/32   33.111.21.15                           0 1001 i
 *>  10.111.3.15/32   33.111.21.15             0             0 1001 i
 *   10.112.3.18/32   33.22.21.22                            0 101 520 2042 i
 *>                   44.33.24.24                            0 520 2042 i
     Network          Next Hop            Metric LocPrf Weight Path
 *   44.114.25.0/24   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
 *   44.114.26.0/24   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
 *   44.115.25.0/24   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
R21#
R21#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/32 is subnetted, 9 subnets
B        10.22.22.22 [20/0] via 33.22.21.22, 01:16:31
C        10.33.33.21 is directly connected, Loopback0
B        10.44.44.23 [20/0] via 44.33.24.24, 01:16:08
B        10.44.44.24 [20/0] via 44.33.24.24, 01:16:31
B        10.44.44.25 [20/0] via 44.33.24.24, 01:16:08
B        10.44.44.26 [20/0] via 44.33.24.24, 01:16:08
B        10.111.3.14 [20/0] via 33.111.21.15, 01:16:24
B        10.111.3.15 [20/0] via 33.111.21.15, 01:16:24
B        10.112.3.18 [20/0] via 44.33.24.24, 01:16:08
      33.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        33.22.21.0/24 is directly connected, Ethernet0/1
L        33.22.21.21/32 is directly connected, Ethernet0/1
C        33.111.21.0/24 is directly connected, Ethernet1/0
L        33.111.21.21/32 is directly connected, Ethernet1/0
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        44.33.24.0/24 is directly connected, Ethernet0/0
L        44.33.24.21/32 is directly connected, Ethernet0/0
B        44.114.25.0/24 [20/0] via 44.33.24.24, 01:16:08
B        44.114.26.0/24 [20/0] via 44.33.24.24, 01:16:08
B        44.115.25.0/24 [20/0] via 44.33.24.24, 01:16:08
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
R22#show ip bgp summary
BGP router identifier 10.22.22.22, local AS number 101
BGP table version is 16, main routing table version 16
12 network entries using 1680 bytes of memory
34 path entries using 2720 bytes of memory
15/6 BGP path/bestpath attribute entries using 2160 bytes of memory
12 BGP AS-PATH entries using 336 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 6896 total bytes of memory
BGP activity 12/0 prefixes, 34/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
22.111.22.14    4         1001      88      88       16    0    0 01:11:52       11
33.22.21.21     4          301      88      87       16    0    0 01:11:51       11
44.22.23.23     4          520      87      87       16    0    0 01:11:55       11
R22#
R22#show ip bgp
BGP table version is 16, local router ID is 10.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   0.0.0.0                  0         32768 i
 *   10.33.33.21/32   44.22.23.23                            0 520 301 i
 *                    22.111.22.14                           0 1001 1001 1001 301 i
 *>                   33.22.21.21              0             0 301 i
 *   10.44.44.23/32   22.111.22.14                           0 1001 1001 1001 301 520 i
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23              0             0 520 i
 *   10.44.44.24/32   22.111.22.14                           0 1001 1001 1001 301 520 i
 *>                   44.22.23.23                            0 520 i
 *                    33.22.21.21                            0 301 520 i
 *   10.44.44.25/32   22.111.22.14                           0 1001 1001 1001 301 520 i
     Network          Next Hop            Metric LocPrf Weight Path
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.44.44.26/32   22.111.22.14                           0 1001 1001 1001 301 520 i
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.111.3.14/32   44.22.23.23                            0 520 301 1001 i
 *>                   33.22.21.21                            0 301 1001 i
 *                    22.111.22.14             0             0 1001 1001 1001 i
 *   10.111.3.15/32   44.22.23.23                            0 520 301 1001 i
 *                    22.111.22.14                           0 1001 1001 1001 i
 *>                   33.22.21.21                            0 301 1001 i
 *>  10.112.3.18/32   44.22.23.23                            0 520 2042 i
 *                    22.111.22.14                           0 1001 1001 1001 301 520 2042 i
 *                    33.22.21.21                            0 301 520 2042 i
 *   44.114.25.0/24   22.111.22.14                           0 1001 1001 1001 301 520 i
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.114.26.0/24   22.111.22.14                           0 1001 1001 1001 301 520 i
     Network          Next Hop            Metric LocPrf Weight Path
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.115.25.0/24   22.111.22.14                           0 1001 1001 1001 301 520 i
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
R22#
R22#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/32 is subnetted, 9 subnets
C        10.22.22.22 is directly connected, Loopback0
B        10.33.33.21 [20/0] via 33.22.21.21, 01:11:44
B        10.44.44.23 [20/0] via 44.22.23.23, 01:11:45
B        10.44.44.24 [20/0] via 44.22.23.23, 01:11:23
B        10.44.44.25 [20/0] via 44.22.23.23, 01:11:23
B        10.44.44.26 [20/0] via 44.22.23.23, 01:11:23
B        10.111.3.14 [20/0] via 33.22.21.21, 01:11:13
B        10.111.3.15 [20/0] via 33.22.21.21, 01:11:13
B        10.112.3.18 [20/0] via 44.22.23.23, 00:38:15
      22.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        22.111.22.0/24 is directly connected, Ethernet1/0
L        22.111.22.22/32 is directly connected, Ethernet1/0
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        33.22.21.0/24 is directly connected, Ethernet0/1
L        33.22.21.22/32 is directly connected, Ethernet0/1
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        44.22.23.0/24 is directly connected, Ethernet0/0
L        44.22.23.22/32 is directly connected, Ethernet0/0
B        44.114.25.0/24 [20/0] via 44.22.23.23, 01:11:23
B        44.114.26.0/24 [20/0] via 44.22.23.23, 01:11:23
B        44.115.25.0/24 [20/0] via 44.22.23.23, 01:11:23
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
 network 10.44.44.23 mask 255.255.255.255
 neighbor 10.44.44.25 remote-as 520
 neighbor 10.44.44.25 next-hop-self
 neighbor 44.22.23.22 remote-as 101
!
```

```
R23#show ip bgp summary 
BGP router identifier 10.44.44.23, local AS number 520
BGP table version is 18, main routing table version 18
12 network entries using 1680 bytes of memory
15 path entries using 1200 bytes of memory
8/6 BGP path/bestpath attribute entries using 1152 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4224 total bytes of memory
BGP activity 12/0 prefixes, 16/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.25     4          520      62      62       18    0    0 00:46:02       10
44.22.23.22     4          101      59      58       18    0    0 00:45:57        4
R23#
R23#show ip bgp
BGP table version is 18, local router ID is 10.44.44.23
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.22.23.22              0             0 101 i
 *>i 10.33.33.21/32   10.44.44.24              0    100      0 301 i
 *                    44.22.23.22                            0 101 301 i
 *>  10.44.44.23/32   0.0.0.0                  0         32768 i
 r>i 10.44.44.24/32   10.44.44.24              0    100      0 i
 r>i 10.44.44.25/32   10.44.44.25              0    100      0 i
 r>i 10.44.44.26/32   10.44.44.26              0    100      0 i
 *>i 10.111.3.14/32   10.44.44.24              0    100      0 301 1001 i
 *                    44.22.23.22                            0 101 301 1001 i
 *>i 10.111.3.15/32   10.44.44.24              0    100      0 301 1001 i
 *                    44.22.23.22                            0 101 301 1001 i
 *>i 10.112.3.18/32   10.44.44.26              0    100      0 2042 i
 *>i 44.114.25.0/24   10.44.44.25              0    100      0 i
 *>i 44.114.26.0/24   10.44.44.26              0    100      0 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>i 44.115.25.0/24   10.44.44.25              0    100      0 i
R23#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
B        10.22.22.22/32 [20/0] via 44.22.23.22, 00:45:31
B        10.33.33.21/32 [200/0] via 10.44.44.24, 00:12:02
C        10.44.44.0/30 is directly connected, Ethernet0/1
L        10.44.44.1/32 is directly connected, Ethernet0/1
C        10.44.44.4/30 is directly connected, Ethernet0/2
L        10.44.44.5/32 is directly connected, Ethernet0/2
i L2     10.44.44.8/30 [115/20] via 10.44.44.6, 00:46:37, Ethernet0/2
i L1     10.44.44.12/30 [115/20] via 10.44.44.2, 00:46:37, Ethernet0/1
C        10.44.44.23/32 is directly connected, Loopback0
i L2     10.44.44.24/32 [115/20] via 10.44.44.6, 00:46:37, Ethernet0/2
i L1     10.44.44.25/32 [115/20] via 10.44.44.2, 00:46:37, Ethernet0/1
i L2     10.44.44.26/32 [115/30] via 10.44.44.6, 00:46:27, Ethernet0/2
                        [115/30] via 10.44.44.2, 00:46:27, Ethernet0/1
B        10.111.3.14/32 [200/0] via 10.44.44.24, 00:12:02
B        10.111.3.15/32 [200/0] via 10.44.44.24, 00:12:02
B        10.112.3.18/32 [200/0] via 10.44.44.26, 00:12:00
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        44.22.23.0/24 is directly connected, Ethernet0/0
L        44.22.23.23/32 is directly connected, Ethernet0/0
B        44.114.25.0/24 [200/0] via 10.44.44.25, 00:45:36
B        44.114.26.0/24 [200/0] via 10.44.44.26, 00:45:33
B        44.115.25.0/24 [200/0] via 10.44.44.25, 00:45:36
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
 network 10.44.44.24 mask 255.255.255.255
 neighbor 10.44.44.25 remote-as 520
 neighbor 10.44.44.25 next-hop-self
 neighbor 44.33.24.21 remote-as 301
 neighbor 44.112.24.18 remote-as 2042
!
```

```
R24#show ip bgp summary
BGP router identifier 10.44.44.24, local AS number 520
BGP table version is 14, main routing table version 14
12 network entries using 1680 bytes of memory
14 path entries using 1120 bytes of memory
8/6 BGP path/bestpath attribute entries using 1152 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4120 total bytes of memory
BGP activity 12/0 prefixes, 14/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.25     4          520      66      67       14    0    0 00:48:50        8
44.33.24.21     4          301      63      62       14    0    0 00:48:46        4
44.112.24.18    4         2042      62      63       14    0    0 00:48:52        1
R24#
R24#show ip bgp        
BGP table version is 14, local router ID is 10.44.44.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.22.22.22/32   10.44.44.23              0    100      0 101 i
 *                    44.33.24.21                            0 301 101 i
 *>  10.33.33.21/32   44.33.24.21              0             0 301 i
 r>i 10.44.44.23/32   10.44.44.23              0    100      0 i
 *>  10.44.44.24/32   0.0.0.0                  0         32768 i
 r>i 10.44.44.25/32   10.44.44.25              0    100      0 i
 r>i 10.44.44.26/32   10.44.44.26              0    100      0 i
 *>  10.111.3.14/32   44.33.24.21                            0 301 1001 i
 *>  10.111.3.15/32   44.33.24.21                            0 301 1001 i
 * i 10.112.3.18/32   10.44.44.26              0    100      0 2042 i
 *>                   44.112.24.18             0             0 2042 i
 *>i 44.114.25.0/24   10.44.44.25              0    100      0 i
 *>i 44.114.26.0/24   10.44.44.26              0    100      0 i
 *>i 44.115.25.0/24   10.44.44.25              0    100      0 i
R24#
R24#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
B        10.22.22.22/32 [200/0] via 10.44.44.23, 00:16:27
B        10.33.33.21/32 [20/0] via 44.33.24.21, 00:49:58
i L2     10.44.44.0/30 [115/20] via 10.44.44.5, 00:51:06, Ethernet0/2
C        10.44.44.4/30 is directly connected, Ethernet0/2
L        10.44.44.6/32 is directly connected, Ethernet0/2
C        10.44.44.8/30 is directly connected, Ethernet0/1
L        10.44.44.9/32 is directly connected, Ethernet0/1
i L2     10.44.44.12/30 [115/20] via 10.44.44.10, 00:51:06, Ethernet0/1
i L2     10.44.44.23/32 [115/20] via 10.44.44.5, 00:51:06, Ethernet0/2
C        10.44.44.24/32 is directly connected, Loopback0
i L2     10.44.44.25/32 [115/30] via 10.44.44.10, 00:51:04, Ethernet0/1
                        [115/30] via 10.44.44.5, 00:51:04, Ethernet0/2
i L2     10.44.44.26/32 [115/20] via 10.44.44.10, 00:51:06, Ethernet0/1
B        10.111.3.14/32 [20/0] via 44.33.24.21, 00:49:27
B        10.111.3.15/32 [20/0] via 44.33.24.21, 00:49:27
B        10.112.3.18/32 [20/0] via 44.112.24.18, 00:50:00
      44.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
C        44.33.24.0/24 is directly connected, Ethernet0/0
L        44.33.24.24/32 is directly connected, Ethernet0/0
C        44.112.24.0/24 is directly connected, Ethernet1/0
L        44.112.24.24/32 is directly connected, Ethernet1/0
B        44.114.25.0/24 [200/0] via 10.44.44.25, 00:50:03
B        44.114.26.0/24 [200/0] via 10.44.44.26, 00:50:00
B        44.115.25.0/24 [200/0] via 10.44.44.25, 00:50:03
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
 network 10.44.44.25 mask 255.255.255.255
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
BGP table version is 15, main routing table version 15
12 network entries using 1680 bytes of memory
13 path entries using 1040 bytes of memory
6/6 BGP path/bestpath attribute entries using 864 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3680 total bytes of memory
BGP activity 12/0 prefixes, 17/4 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.23     4          520      66      66       15    0    0 00:49:15        2
10.44.44.24     4          520      67      66       15    0    0 00:49:13        5
10.44.44.26     4          520      64      67       15    0    0 00:49:11        3
R25#
R25#show ip bgp        
BGP table version is 15, local router ID is 10.44.44.25
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.22.22.22/32   10.44.44.23              0    100      0 101 i
 *>i 10.33.33.21/32   10.44.44.24              0    100      0 301 i
 r>i 10.44.44.23/32   10.44.44.23              0    100      0 i
 r>i 10.44.44.24/32   10.44.44.24              0    100      0 i
 *>  10.44.44.25/32   0.0.0.0                  0         32768 i
 r>i 10.44.44.26/32   10.44.44.26              0    100      0 i
 *>i 10.111.3.14/32   10.44.44.24              0    100      0 301 1001 i
 *>i 10.111.3.15/32   10.44.44.24              0    100      0 301 1001 i
 *>i 10.112.3.18/32   10.44.44.26              0    100      0 2042 i
 * i                  10.44.44.24              0    100      0 2042 i
 *>  44.114.25.0/24   0.0.0.0                  0         32768 i
 *>i 44.114.26.0/24   10.44.44.26              0    100      0 i
 *>  44.115.25.0/24   0.0.0.0                  0         32768 i
R25#
R25#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 17 subnets, 3 masks
B        10.22.22.22/32 [200/0] via 10.44.44.23, 00:18:22
B        10.33.33.21/32 [200/0] via 10.44.44.24, 00:18:23
C        10.44.44.0/30 is directly connected, Ethernet0/1
L        10.44.44.2/32 is directly connected, Ethernet0/1
i L1     10.44.44.4/30 [115/20] via 10.44.44.1, 00:53:01, Ethernet0/1
i L2     10.44.44.8/30 [115/20] via 10.44.44.14, 00:53:01, Ethernet0/2
C        10.44.44.12/30 is directly connected, Ethernet0/2
L        10.44.44.13/32 is directly connected, Ethernet0/2
i L1     10.44.44.23/32 [115/20] via 10.44.44.1, 00:53:01, Ethernet0/1
i L2     10.44.44.24/32 [115/30] via 10.44.44.14, 00:53:01, Ethernet0/2
                        [115/30] via 10.44.44.1, 00:53:01, Ethernet0/1
C        10.44.44.25/32 is directly connected, Loopback0
i L2     10.44.44.26/32 [115/20] via 10.44.44.14, 00:53:01, Ethernet0/2
B        10.111.3.14/32 [200/0] via 10.44.44.24, 00:18:23
B        10.111.3.15/32 [200/0] via 10.44.44.24, 00:18:23
S        10.111.4.64/26 [1/0] via 44.114.25.28
S        10.111.4.128/26 [1/0] via 44.114.25.28
B        10.112.3.18/32 [200/0] via 10.44.44.26, 00:18:22
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        44.114.25.0/24 is directly connected, Ethernet1/1
L        44.114.25.25/32 is directly connected, Ethernet1/1
B        44.114.26.0/24 [200/0] via 10.44.44.26, 00:51:55
C        44.115.25.0/24 is directly connected, Ethernet1/0
L        44.115.25.25/32 is directly connected, Ethernet1/0
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
 network 10.44.44.26 mask 255.255.255.255
 network 44.114.26.0 mask 255.255.255.0
 neighbor 10.44.44.25 remote-as 520
 neighbor 10.44.44.25 next-hop-self
 neighbor 44.112.26.18 remote-as 2042
!
```

```
R26#show ip bgp summary
BGP router identifier 10.44.44.26, local AS number 520
BGP table version is 14, main routing table version 14
12 network entries using 1680 bytes of memory
12 path entries using 960 bytes of memory
6/6 BGP path/bestpath attribute entries using 864 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3648 total bytes of memory
BGP activity 12/0 prefixes, 13/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.44.44.25     4          520      67      64       14    0    0 00:49:02        9
44.112.26.18    4         2042      62      62       14    0    0 00:49:00        1
R26#                   
R26#show ip bgp        
BGP table version is 14, local router ID is 10.44.44.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.22.22.22/32   10.44.44.23              0    100      0 101 i
 *>i 10.33.33.21/32   10.44.44.24              0    100      0 301 i
 r>i 10.44.44.23/32   10.44.44.23              0    100      0 i
 r>i 10.44.44.24/32   10.44.44.24              0    100      0 i
 r>i 10.44.44.25/32   10.44.44.25              0    100      0 i
 *>  10.44.44.26/32   0.0.0.0                  0         32768 i
 *>i 10.111.3.14/32   10.44.44.24              0    100      0 301 1001 i
 *>i 10.111.3.15/32   10.44.44.24              0    100      0 301 1001 i
 *>  10.112.3.18/32   44.112.26.18             0             0 2042 i
 *>i 44.114.25.0/24   10.44.44.25              0    100      0 i
 *>  44.114.26.0/24   0.0.0.0                  0         32768 i
 *>i 44.115.25.0/24   10.44.44.25              0    100      0 i
R26#
R26#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 17 subnets, 3 masks
B        10.22.22.22/32 [200/0] via 10.44.44.23, 00:16:50
B        10.33.33.21/32 [200/0] via 10.44.44.24, 00:16:52
i L2     10.44.44.0/30 [115/20] via 10.44.44.13, 00:51:30, Ethernet0/2
i L2     10.44.44.4/30 [115/20] via 10.44.44.9, 00:51:30, Ethernet0/1
C        10.44.44.8/30 is directly connected, Ethernet0/1
L        10.44.44.10/32 is directly connected, Ethernet0/1
C        10.44.44.12/30 is directly connected, Ethernet0/2
L        10.44.44.14/32 is directly connected, Ethernet0/2
i L2     10.44.44.23/32 [115/30] via 10.44.44.13, 00:51:20, Ethernet0/2
                        [115/30] via 10.44.44.9, 00:51:20, Ethernet0/1
i L2     10.44.44.24/32 [115/20] via 10.44.44.9, 00:51:30, Ethernet0/1
i L2     10.44.44.25/32 [115/20] via 10.44.44.13, 00:51:30, Ethernet0/2
C        10.44.44.26/32 is directly connected, Loopback0
B        10.111.3.14/32 [200/0] via 10.44.44.24, 00:16:52
B        10.111.3.15/32 [200/0] via 10.44.44.24, 00:16:52
S        10.111.4.64/26 [1/0] via 44.114.26.28
S        10.111.4.128/26 [1/0] via 44.114.26.28
B        10.112.3.18/32 [20/0] via 44.112.26.18, 00:50:24
      44.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C        44.112.26.0/24 is directly connected, Ethernet1/1
L        44.112.26.26/32 is directly connected, Ethernet1/1
B        44.114.25.0/24 [200/0] via 10.44.44.25, 00:50:24
C        44.114.26.0/24 is directly connected, Ethernet1/0
L        44.114.26.26/32 is directly connected, Ethernet1/0
B        44.115.25.0/24 [200/0] via 10.44.44.25, 00:50:24
```

