# Лабораторная работа 09
+ ## BGP. Основы
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw09/Network_topology_BGP1.png?raw=true)

## Цели:
+ ### Настройка BGP между автономными системами
+ ### Организовать доступность между офисами Москва и С.-Петербург

### Этапы:
+ #### Шаг 1: Настройка eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
+ #### Шаг 2: Настройка eBGP между провайдерами Киторн и Ламас
+ #### Шаг 3: Настройка eBGP между Ламас и Триада
+ #### Шаг 4: Настройка eBGP между офисом С.-Петербург и провайдером Триада

#### R14:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
interface Ethernet1/0
 description to R22 Kitorn
 ip address 22.111.22.14 255.255.255.0
!
router bgp 1001
 bgp router-id 10.111.3.14
 bgp log-neighbor-changes
 network 22.111.22.0 mask 255.255.255.0
 neighbor 22.111.22.22 remote-as 101
!
```

```
R14#show ip route bgp
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

      10.0.0.0/8 is variably subnetted, 22 subnets, 2 masks
B        10.22.22.22/32 [20/0] via 22.111.22.22, 00:26:26
B        10.33.33.21/32 [20/0] via 22.111.22.22, 00:23:41
      33.0.0.0/24 is subnetted, 2 subnets
B        33.22.21.0 [20/0] via 22.111.22.22, 00:25:07
B        33.111.21.0 [20/0] via 22.111.22.22, 00:22:33
      44.0.0.0/24 is subnetted, 4 subnets
B        44.22.23.0 [20/0] via 22.111.22.22, 00:24:37
B        44.33.24.0 [20/0] via 22.111.22.22, 00:21:19
B        44.112.24.0 [20/0] via 22.111.22.22, 00:11:44
B        44.112.26.0 [20/0] via 22.111.22.22, 00:09:16
R14#
R14#show ip bgp
BGP table version is 12, local router ID is 10.111.3.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   22.111.22.22             0             0 101 i
 *>  10.33.33.21/32   22.111.22.22                           0 101 301 i
 *   22.111.22.0/24   22.111.22.22             0             0 101 i
 *>                   0.0.0.0                  0         32768 i
 *>  33.22.21.0/24    22.111.22.22             0             0 101 i
 *>  33.111.21.0/24   22.111.22.22                           0 101 301 i
 *>  44.22.23.0/24    22.111.22.22             0             0 101 i
 *>  44.33.24.0/24    22.111.22.22                           0 101 301 i
 *>  44.112.24.0/24   22.111.22.22                           0 101 301 520 i
 *>  44.112.26.0/24   22.111.22.22                           0 101 301 520 2042 i
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
interface Ethernet1/0
 description to R21 Lamas
 ip address 33.111.21.15 255.255.255.0
!
router bgp 1001
 bgp router-id 10.111.3.15
 bgp log-neighbor-changes
 network 33.111.21.0 mask 255.255.255.0
 neighbor 33.111.21.21 remote-as 301
!
```

```
R15#show ip route bgp
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

      10.0.0.0/8 is variably subnetted, 22 subnets, 2 masks
B        10.22.22.22/32 [20/0] via 33.111.21.21, 00:26:26
B        10.33.33.21/32 [20/0] via 33.111.21.21, 00:23:41
      22.0.0.0/24 is subnetted, 1 subnets
B        22.111.22.0 [20/0] via 33.111.21.21, 00:25:37
      33.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
B        33.22.21.0/24 [20/0] via 33.111.21.21, 00:22:03
      44.0.0.0/24 is subnetted, 4 subnets
B        44.22.23.0 [20/0] via 33.111.21.21, 00:24:36
B        44.33.24.0 [20/0] via 33.111.21.21, 00:21:19
B        44.112.24.0 [20/0] via 33.111.21.21, 00:11:44
B        44.112.26.0 [20/0] via 33.111.21.21, 00:09:15
R15#
R15#show ip bgp
BGP table version is 13, local router ID is 10.111.3.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   33.111.21.21                           0 301 101 i
 *>  10.33.33.21/32   33.111.21.21             0             0 301 i
 *>  22.111.22.0/24   33.111.21.21                           0 301 101 i
 *>  33.22.21.0/24    33.111.21.21             0             0 301 i
 *   33.111.21.0/24   33.111.21.21             0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *>  44.22.23.0/24    33.111.21.21                           0 301 101 i
 *>  44.33.24.0/24    33.111.21.21             0             0 301 i
 *>  44.112.24.0/24   33.111.21.21                           0 301 520 i
 *>  44.112.26.0/24   33.111.21.21                           0 301 520 2042 i
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
 network 44.112.24.0 mask 255.255.255.0
 network 44.112.26.0 mask 255.255.255.0
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.26.26 remote-as 520
!
```

```
R18#show ip route bgp
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

      10.0.0.0/8 is variably subnetted, 14 subnets, 3 masks
B        10.22.22.22/32 [20/0] via 44.112.24.24, 00:26:26
B        10.33.33.21/32 [20/0] via 44.112.24.24, 00:23:41
      22.0.0.0/24 is subnetted, 1 subnets
B        22.111.22.0 [20/0] via 44.112.24.24, 00:25:37
      33.0.0.0/24 is subnetted, 2 subnets
B        33.22.21.0 [20/0] via 44.112.24.24, 00:22:03
B        33.111.21.0 [20/0] via 44.112.24.24, 00:22:33
      44.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
B        44.22.23.0/24 [20/0] via 44.112.24.24, 00:24:36
B        44.33.24.0/24 [20/0] via 44.112.24.24, 00:12:13
R18#
R18#show ip bgp
BGP table version is 20, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.112.24.24                           0 520 301 101 i
 *>  10.33.33.21/32   44.112.24.24                           0 520 301 i
 *>  22.111.22.0/24   44.112.24.24                           0 520 301 101 i
 *>  33.22.21.0/24    44.112.24.24                           0 520 301 i
 *>  33.111.21.0/24   44.112.24.24                           0 520 301 i
 *>  44.22.23.0/24    44.112.24.24                           0 520 301 101 i
 *>  44.33.24.0/24    44.112.24.24             0             0 520 i
 *>  44.112.24.0/24   0.0.0.0                  0         32768 i
 *                    44.112.24.24             0             0 520 i
 *>  44.112.26.0/24   0.0.0.0                  0         32768 i
 *                    44.112.26.26             0             0 520 i
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
 network 33.22.21.0 mask 255.255.255.0
 network 33.111.21.0 mask 255.255.255.0
 network 44.33.24.0 mask 255.255.255.0
 neighbor 33.22.21.22 remote-as 101
 neighbor 33.111.21.15 remote-as 1001
 neighbor 44.33.24.24 remote-as 520
!
```

```
R21#show ip route bgp
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

      10.0.0.0/32 is subnetted, 2 subnets
B        10.22.22.22 [20/0] via 33.22.21.22, 00:26:27
      22.0.0.0/24 is subnetted, 1 subnets
B        22.111.22.0 [20/0] via 33.22.21.22, 00:25:38
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
B        44.22.23.0/24 [20/0] via 33.22.21.22, 00:24:37
B        44.112.24.0/24 [20/0] via 44.33.24.24, 00:11:44
B        44.112.26.0/24 [20/0] via 44.33.24.24, 00:09:16
R21# 
R21#show ip bgp
BGP table version is 17, local router ID is 10.33.33.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   33.22.21.22              0             0 101 i
 *>  10.33.33.21/32   0.0.0.0                  0         32768 i
 *>  22.111.22.0/24   33.22.21.22              0             0 101 i
 *>  33.22.21.0/24    0.0.0.0                  0         32768 i
 *                    33.22.21.22              0             0 101 i
 *>  33.111.21.0/24   0.0.0.0                  0         32768 i
 *                    33.111.21.15             0             0 1001 i
 *>  44.22.23.0/24    33.22.21.22              0             0 101 i
 *   44.33.24.0/24    44.33.24.24              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  44.112.24.0/24   44.33.24.24              0             0 520 i
 *>  44.112.26.0/24   44.33.24.24                            0 520 2042 i
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
 network 22.111.22.0 mask 255.255.255.0
 network 33.22.21.0 mask 255.255.255.0
 network 44.22.23.0 mask 255.255.255.0
 neighbor 22.111.22.14 remote-as 1001
 neighbor 33.22.21.21 remote-as 301
 neighbor 44.22.23.23 remote-as 520
!
```

```
R22#show ip route bgp
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

      10.0.0.0/32 is subnetted, 2 subnets
B        10.33.33.21 [20/0] via 33.22.21.21, 00:23:42
      33.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
B        33.111.21.0/24 [20/0] via 33.22.21.21, 00:22:33
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
B        44.33.24.0/24 [20/0] via 33.22.21.21, 00:21:20
B        44.112.24.0/24 [20/0] via 33.22.21.21, 00:11:44
B        44.112.26.0/24 [20/0] via 33.22.21.21, 00:09:16
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
 *>  10.33.33.21/32   33.22.21.21              0             0 301 i
 *>  22.111.22.0/24   0.0.0.0                  0         32768 i
 *                    22.111.22.14             0             0 1001 i
 *   33.22.21.0/24    33.22.21.21              0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *>  33.111.21.0/24   33.22.21.21              0             0 301 i
 *   44.22.23.0/24    44.22.23.23              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  44.33.24.0/24    33.22.21.21              0             0 301 i
 *>  44.112.24.0/24   33.22.21.21                            0 301 520 i
 *>  44.112.26.0/24   33.22.21.21                            0 301 520 2042 i
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
interface Ethernet0/0
 description to R22 Kitorn
 ip address 44.22.23.23 255.255.255.0
!
router bgp 520
 bgp router-id 10.44.44.23
 bgp log-neighbor-changes
 network 44.22.23.0 mask 255.255.255.0
 neighbor 44.22.23.22 remote-as 101
!
```

```
R23#show ip route bgp
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

      10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
B        10.22.22.22/32 [20/0] via 44.22.23.22, 00:26:26
B        10.33.33.21/32 [20/0] via 44.22.23.22, 00:23:41
      22.0.0.0/24 is subnetted, 1 subnets
B        22.111.22.0 [20/0] via 44.22.23.22, 00:25:37
      33.0.0.0/24 is subnetted, 2 subnets
B        33.22.21.0 [20/0] via 44.22.23.22, 00:25:07
B        33.111.21.0 [20/0] via 44.22.23.22, 00:22:33
      44.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
B        44.33.24.0/24 [20/0] via 44.22.23.22, 00:21:19
R23# 
R23#show ip bgp
BGP table version is 15, local router ID is 10.44.44.23
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.22.23.22              0             0 101 i
 *>  10.33.33.21/32   44.22.23.22                            0 101 301 i
 *>  22.111.22.0/24   44.22.23.22              0             0 101 i
 *>  33.22.21.0/24    44.22.23.22              0             0 101 i
 *>  33.111.21.0/24   44.22.23.22                            0 101 301 i
 *>  44.22.23.0/24    0.0.0.0                  0         32768 i
 *                    44.22.23.22              0             0 101 i
 *>  44.33.24.0/24    44.22.23.22                            0 101 301 i
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
interface Ethernet0/0
 description to R21 Lamas
 ip address 44.33.24.24 255.255.255.0
!
interface Ethernet1/0
 description to R18 SPb
 ip address 44.112.24.24 255.255.255.0
!
router bgp 520
 bgp router-id 10.44.44.24
 bgp log-neighbor-changes
 network 44.33.24.0 mask 255.255.255.0
 network 44.112.24.0 mask 255.255.255.0
 neighbor 44.33.24.21 remote-as 301
 neighbor 44.112.24.18 remote-as 2042
!
```

```
R24#show ip route bgp
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

      10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
B        10.22.22.22/32 [20/0] via 44.33.24.21, 00:26:26
B        10.33.33.21/32 [20/0] via 44.33.24.21, 00:23:41
      22.0.0.0/24 is subnetted, 1 subnets
B        22.111.22.0 [20/0] via 44.33.24.21, 00:25:37
      33.0.0.0/24 is subnetted, 2 subnets
B        33.22.21.0 [20/0] via 44.33.24.21, 00:22:03
B        33.111.21.0 [20/0] via 44.33.24.21, 00:22:33
      44.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
B        44.22.23.0/24 [20/0] via 44.33.24.21, 00:24:36
B        44.112.26.0/24 [20/0] via 44.112.24.18, 00:09:15
R24#
R24#show ip bgp
BGP table version is 18, local router ID is 10.44.44.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.33.24.21                            0 301 101 i
 *>  10.33.33.21/32   44.33.24.21              0             0 301 i
 *>  22.111.22.0/24   44.33.24.21                            0 301 101 i
 *>  33.22.21.0/24    44.33.24.21              0             0 301 i
 *>  33.111.21.0/24   44.33.24.21              0             0 301 i
 *>  44.22.23.0/24    44.33.24.21                            0 301 101 i
 *>  44.33.24.0/24    0.0.0.0                  0         32768 i
 *                    44.33.24.21              0             0 301 i
 *   44.112.24.0/24   44.112.24.18             0             0 2042 i
 *>                   0.0.0.0                  0         32768 i
 *>  44.112.26.0/24   44.112.24.18             0             0 2042 i
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
interface Ethernet1/1
 description to R18 SPb
 ip address 44.112.26.26 255.255.255.0
!
router bgp 520
 bgp router-id 10.44.44.26
 bgp log-neighbor-changes
 network 44.112.26.0 mask 255.255.255.0
 neighbor 44.112.26.18 remote-as 2042
!
```

```
R26#show ip route bgp
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

      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
B        44.112.24.0/24 [20/0] via 44.112.26.18, 00:09:47
R26# 
R26#show ip bgp
BGP table version is 3, local router ID is 10.44.44.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  44.112.24.0/24   44.112.26.18             0             0 2042 i
 *   44.112.26.0/24   44.112.26.18             0             0 2042 i
 *>                   0.0.0.0                  0         32768 i
```

+ #### Шаг 5: Организация IP-доступности между пограничным роутерами в офисах Москва и С.-Петербург

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