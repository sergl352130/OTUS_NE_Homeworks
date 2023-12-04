# Лабораторная работа 11
+ ## BGP. Фильтрация
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw09/Network_topology_BGP1.png?raw=true)

## Цели:
+ ### Настроить фильтрацию для офиса Москва
+ ### Настроить фильтрацию для офиса С.-Петербург

### Этапы:
+ #### Шаг 1: Настроить фильтрацию для офиса Москва так, чтобы исключить транзитный трафик (As-path)
+ #### Шаг 2: Настроить фильтрацию для офиса С.-Петербург так, чтобы исключить транзитный трафик (Prefix-list)
+ #### Шаг 3: Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию
+ #### Шаг 4: Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург
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
!
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
!
route-map R22-Kitorn-OUT permit 10
 match as-path 1
 set as-path prepend 1001 1001
!
```

```
R14#show ip bgp neighbors 22.111.22.22 advertised-routes 
BGP table version is 21, local router ID is 10.111.3.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.22.22.22/32   10.111.3.15              0   1000      0 301 101 i
 *>i 10.33.33.21/32   10.111.3.15              0   1000      0 301 i
 *>i 10.44.44.23/32   10.111.3.15              0   1000      0 301 520 i
 *>i 10.44.44.24/32   10.111.3.15              0   1000      0 301 520 i
 *>i 10.44.44.25/32   10.111.3.15              0   1000      0 301 520 i
 *>i 10.44.44.26/32   10.111.3.15              0   1000      0 301 520 i
 *>  10.111.3.14/32   0.0.0.0                  0         32768 i
 r>i 10.111.3.15/32   10.111.3.15              0    100      0 i
 *>i 10.112.3.18/32   10.111.3.15              0   1000      0 301 520 2042 i
 *>i 44.114.25.0/24   10.111.3.15              0   1000      0 301 520 i
 *>i 44.114.26.0/24   10.111.3.15              0   1000      0 301 520 i
 *>i 44.115.25.0/24   10.111.3.15              0   1000      0 301 520 i

Total number of prefixes 12
```

##### После применения фильтрации транзитного трафика на R14:

```                                                   
R14#show ip bgp neighbors 22.111.22.22 advertised-routes 
BGP table version is 21, local router ID is 10.111.3.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.111.3.14/32   0.0.0.0                  0         32768 i
 r>i 10.111.3.15/32   10.111.3.15              0    100      0 i

Total number of prefixes 2
```

##### После включения default route и применения фильтрации исходящих маршрутов на R22:

```       
R14#show ip bgp
BGP table version is 22, local router ID is 10.111.3.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          22.111.22.22                           0 101 i         (!!!)
 *>i 10.22.22.22/32   10.111.3.15              0   1000      0 301 101 i
 *>i 10.33.33.21/32   10.111.3.15              0   1000      0 301 i
 *>i 10.44.44.23/32   10.111.3.15              0   1000      0 301 520 i
 *>i 10.44.44.24/32   10.111.3.15              0   1000      0 301 520 i
 *>i 10.44.44.25/32   10.111.3.15              0   1000      0 301 520 i
 *>i 10.44.44.26/32   10.111.3.15              0   1000      0 301 520 i
 *>  10.111.3.14/32   0.0.0.0                  0         32768 i
 r>i 10.111.3.15/32   10.111.3.15              0    100      0 i
 *>i 10.112.3.18/32   10.111.3.15              0   1000      0 301 520 2042 i
 *>i 44.114.25.0/24   10.111.3.15              0   1000      0 301 520 i
 *>i 44.114.26.0/24   10.111.3.15              0   1000      0 301 520 i
 *>i 44.115.25.0/24   10.111.3.15              0   1000      0 301 520 i
```

##### После применения всех правил фильтрации у провайдеров Киторн и Ламас:

```       
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

Gateway of last resort is 10.111.3.15 to network 0.0.0.0

B*    0.0.0.0/0 [200/0] via 10.111.3.15, 05:12:16
      10.0.0.0/8 is variably subnetted, 22 subnets, 2 masks
O IA     10.111.2.0/30 [110/20] via 10.111.2.9, 05:13:18, Ethernet0/0
O IA     10.111.2.4/30 [110/20] via 10.111.2.21, 05:13:18, Ethernet0/1
C        10.111.2.8/30 is directly connected, Ethernet0/0
L        10.111.2.10/32 is directly connected, Ethernet0/0
O        10.111.2.12/30 [110/20] via 10.111.2.26, 05:13:18, Ethernet0/2
                        [110/20] via 10.111.2.9, 05:13:18, Ethernet0/0
O        10.111.2.16/30 [110/20] via 10.111.2.26, 05:13:18, Ethernet0/2
                        [110/20] via 10.111.2.21, 05:13:18, Ethernet0/1
C        10.111.2.20/30 is directly connected, Ethernet0/1
L        10.111.2.22/32 is directly connected, Ethernet0/1
C        10.111.2.24/30 is directly connected, Ethernet0/2
L        10.111.2.25/32 is directly connected, Ethernet0/2
C        10.111.2.28/30 is directly connected, Ethernet0/3
L        10.111.2.29/32 is directly connected, Ethernet0/3
O IA     10.111.2.32/30 [110/20] via 10.111.2.26, 05:13:18, Ethernet0/2
O IA     10.111.3.4/32 [110/21] via 10.111.2.9, 05:13:17, Ethernet0/0
O IA     10.111.3.5/32 [110/21] via 10.111.2.21, 05:13:18, Ethernet0/1
O        10.111.3.12/32 [110/11] via 10.111.2.9, 05:13:18, Ethernet0/0
O        10.111.3.13/32 [110/11] via 10.111.2.21, 05:13:18, Ethernet0/1
C        10.111.3.14/32 is directly connected, Loopback0
O        10.111.3.15/32 [110/11] via 10.111.2.26, 05:13:18, Ethernet0/2
O        10.111.3.19/32 [110/11] via 10.111.2.30, 05:13:18, Ethernet0/3
O IA     10.111.3.20/32 [110/21] via 10.111.2.26, 05:13:17, Ethernet0/2
B        10.112.3.18/32 [200/0] via 10.111.3.15, 05:12:03
      22.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        22.111.22.0/24 is directly connected, Ethernet1/0
L        22.111.22.14/32 is directly connected, Ethernet1/0
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
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
!
ip prefix-list NOarea101 seq 5 deny 10.111.2.28/30
ip prefix-list NOarea101 seq 10 deny 10.111.3.19/32
ip prefix-list NOarea101 seq 15 permit 0.0.0.0/0 le 32
!
route-map R21-Lamas-IN permit 10
 set local-preference 1000
!
route-map R21-Lamas-OUT permit 10
 match as-path 1
!
```

```
R15#show ip bgp neighbors 33.111.21.21 advertised-routes
BGP table version is 20, local router ID is 10.111.3.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 10.111.3.14/32   10.111.3.14              0    100      0 i
 *>  10.111.3.15/32   0.0.0.0                  0         32768 i

Total number of prefixes 2 
R15#show ip bgp                                         
BGP table version is 20, local router ID is 10.111.3.15
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

      10.0.0.0/8 is variably subnetted, 28 subnets, 2 masks
B        10.22.22.22/32 [20/0] via 33.111.21.21, 05:14:09
B        10.33.33.21/32 [20/0] via 33.111.21.21, 05:14:09
B        10.44.44.23/32 [20/0] via 33.111.21.21, 05:13:15
B        10.44.44.24/32 [20/0] via 33.111.21.21, 05:13:46
B        10.44.44.25/32 [20/0] via 33.111.21.21, 05:13:15
B        10.44.44.26/32 [20/0] via 33.111.21.21, 05:13:15
O IA     10.111.2.0/30 [110/20] via 10.111.2.13, 05:15:06, Ethernet0/1
O IA     10.111.2.4/30 [110/20] via 10.111.2.17, 05:15:06, Ethernet0/0
O        10.111.2.8/30 [110/20] via 10.111.2.25, 05:15:06, Ethernet0/2
                       [110/20] via 10.111.2.13, 05:15:06, Ethernet0/1
C        10.111.2.12/30 is directly connected, Ethernet0/1
L        10.111.2.14/32 is directly connected, Ethernet0/1
C        10.111.2.16/30 is directly connected, Ethernet0/0
L        10.111.2.18/32 is directly connected, Ethernet0/0
O        10.111.2.20/30 [110/20] via 10.111.2.25, 05:15:06, Ethernet0/2
                        [110/20] via 10.111.2.17, 05:15:06, Ethernet0/0
C        10.111.2.24/30 is directly connected, Ethernet0/2
L        10.111.2.26/32 is directly connected, Ethernet0/2
O IA     10.111.2.28/30 [110/20] via 10.111.2.25, 05:15:06, Ethernet0/2
C        10.111.2.32/30 is directly connected, Ethernet0/3
L        10.111.2.33/32 is directly connected, Ethernet0/3
O IA     10.111.3.4/32 [110/21] via 10.111.2.13, 05:15:06, Ethernet0/1
O IA     10.111.3.5/32 [110/21] via 10.111.2.17, 05:15:06, Ethernet0/0
O        10.111.3.12/32 [110/11] via 10.111.2.13, 05:15:06, Ethernet0/1
O        10.111.3.13/32 [110/11] via 10.111.2.17, 05:15:06, Ethernet0/0
O        10.111.3.14/32 [110/11] via 10.111.2.25, 05:15:06, Ethernet0/2
C        10.111.3.15/32 is directly connected, Loopback0
O IA     10.111.3.19/32 [110/21] via 10.111.2.25, 05:15:06, Ethernet0/2
O        10.111.3.20/32 [110/11] via 10.111.2.34, 05:15:06, Ethernet0/3
B        10.112.3.18/32 [20/0] via 33.111.21.21, 05:13:15
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        33.111.21.0/24 is directly connected, Ethernet1/0
L        33.111.21.15/32 is directly connected, Ethernet1/0
      44.0.0.0/24 is subnetted, 3 subnets
B        44.114.25.0 [20/0] via 33.111.21.21, 05:13:15
B        44.114.26.0 [20/0] via 33.111.21.21, 05:13:15
B        44.115.25.0 [20/0] via 33.111.21.21, 05:13:15
```

##### После включения default route и применения фильтрации исходящих маршрутов на R21:

```       
R15#show ip bgp
BGP table version is 31, local router ID is 10.111.3.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          33.111.21.21                 1000      0 301 i
 r>i 10.111.3.14/32   10.111.3.14              0    100      0 i
 *>  10.111.3.15/32   0.0.0.0                  0         32768 i
 *>  10.112.3.18/32   33.111.21.21                 1000      0 301 520 2042 i
```

##### После применения всех правил фильтрации у провайдеров Киторн и Ламас:

```       
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

Gateway of last resort is 33.111.21.21 to network 0.0.0.0

B*    0.0.0.0/0 [20/0] via 33.111.21.21, 05:10:37
      10.0.0.0/8 is variably subnetted, 22 subnets, 2 masks
O IA     10.111.2.0/30 [110/20] via 10.111.2.13, 05:11:38, Ethernet0/1
O IA     10.111.2.4/30 [110/20] via 10.111.2.17, 05:11:38, Ethernet0/0
O        10.111.2.8/30 [110/20] via 10.111.2.25, 05:11:38, Ethernet0/2
                       [110/20] via 10.111.2.13, 05:11:38, Ethernet0/1
C        10.111.2.12/30 is directly connected, Ethernet0/1
L        10.111.2.14/32 is directly connected, Ethernet0/1
C        10.111.2.16/30 is directly connected, Ethernet0/0
L        10.111.2.18/32 is directly connected, Ethernet0/0
O        10.111.2.20/30 [110/20] via 10.111.2.25, 05:11:38, Ethernet0/2
                        [110/20] via 10.111.2.17, 05:11:38, Ethernet0/0
C        10.111.2.24/30 is directly connected, Ethernet0/2
L        10.111.2.26/32 is directly connected, Ethernet0/2
O IA     10.111.2.28/30 [110/20] via 10.111.2.25, 05:11:38, Ethernet0/2
C        10.111.2.32/30 is directly connected, Ethernet0/3
L        10.111.2.33/32 is directly connected, Ethernet0/3
O IA     10.111.3.4/32 [110/21] via 10.111.2.13, 05:11:38, Ethernet0/1
O IA     10.111.3.5/32 [110/21] via 10.111.2.17, 05:11:38, Ethernet0/0
O        10.111.3.12/32 [110/11] via 10.111.2.13, 05:11:38, Ethernet0/1
O        10.111.3.13/32 [110/11] via 10.111.2.17, 05:11:38, Ethernet0/0
O        10.111.3.14/32 [110/11] via 10.111.2.25, 05:11:38, Ethernet0/2
C        10.111.3.15/32 is directly connected, Loopback0
O IA     10.111.3.19/32 [110/21] via 10.111.2.25, 05:11:38, Ethernet0/2
O        10.111.3.20/32 [110/11] via 10.111.2.34, 05:11:38, Ethernet0/3
B        10.112.3.18/32 [20/0] via 33.111.21.21, 05:10:23
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        33.111.21.0/24 is directly connected, Ethernet1/0
L        33.111.21.15/32 is directly connected, Ethernet1/0
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
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
route-map Triada-OUT permit 10
 match ip address prefix-list Triada-OUT
!
```

```
R18#show ip bgp neighbors 44.112.24.24 advertised-routes
BGP table version is 20, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.112.24.24                           0 520 301 101 i
 *>  10.33.33.21/32   44.112.24.24                           0 520 301 i
 *>  10.44.44.23/32   44.112.24.24                           0 520 i
 *>  10.44.44.24/32   44.112.24.24             0             0 520 i
 *>  10.44.44.25/32   44.112.24.24                           0 520 i
 *>  10.44.44.26/32   44.112.26.26             0             0 520 i
 *>  10.111.3.14/32   44.112.24.24                           0 520 301 1001 i
 *>  10.111.3.15/32   44.112.24.24                           0 520 301 1001 i
 *>  10.112.3.18/32   0.0.0.0                  0         32768 i
 *>  44.114.25.0/24   44.112.24.24                           0 520 i
 *>  44.114.26.0/24   44.112.26.26             0             0 520 i
 *>  44.115.25.0/24   44.112.24.24                           0 520 i

Total number of prefixes 12 
R18#
R18#show ip bgp neighbors 44.112.26.26 advertised-routes
BGP table version is 20, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   44.112.24.24                           0 520 301 101 i
 *>  10.33.33.21/32   44.112.24.24                           0 520 301 i
 *>  10.44.44.23/32   44.112.24.24                           0 520 i
 *>  10.44.44.24/32   44.112.24.24             0             0 520 i
 *>  10.44.44.25/32   44.112.24.24                           0 520 i
 *>  10.44.44.26/32   44.112.26.26             0             0 520 i
 *>  10.111.3.14/32   44.112.24.24                           0 520 301 1001 i
 *>  10.111.3.15/32   44.112.24.24                           0 520 301 1001 i
 *>  10.112.3.18/32   0.0.0.0                  0         32768 i
 *>  44.114.25.0/24   44.112.24.24                           0 520 i
 *>  44.114.26.0/24   44.112.26.26             0             0 520 i
 *>  44.115.25.0/24   44.112.24.24                           0 520 i

Total number of prefixes 12 
```

##### После применения фильтрации транзитного трафика на R18:

```
R18#show ip bgp neighbors 44.112.24.24 advertised-routes
BGP table version is 20, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.112.3.18/32   0.0.0.0                  0         32768 i

Total number of prefixes 1 
R18#
R18#show ip bgp neighbors 44.112.26.26 advertised-routes
BGP table version is 20, local router ID is 10.112.3.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.112.3.18/32   0.0.0.0                  0         32768 i

Total number of prefixes 1 
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
 neighbor 33.111.21.15 default-originate
 neighbor 33.111.21.15 route-map Lamas-OUT out
 neighbor 44.33.24.24 remote-as 520
!
ip as-path access-list 1 permit .*_2042$
ip as-path access-list 1 deny .*
!
route-map Lamas-OUT permit 10
 match as-path 1
!
```

##### После включения default route и применения фильтрации исходящих маршрутов на R21:

```
R21#show ip bgp neighbors 33.111.21.15 advertised-routes
BGP table version is 15, local router ID is 10.33.33.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

Originating default network 0.0.0.0                                          (!!!)

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.112.3.18/32   44.33.24.24                            0 520 2042 i    (!!!)

Total number of prefixes 1 
R21#
R21#show ip bgp
BGP table version is 15, local router ID is 10.33.33.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
     0.0.0.0          0.0.0.0                                0 i             (!!!)
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
 *   44.114.25.0/24   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
 *   44.114.26.0/24   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
 *   44.115.25.0/24   33.22.21.22                            0 101 520 i
 *>                   44.33.24.24                            0 520 i
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
 neighbor 22.111.22.14 default-originate
 neighbor 22.111.22.14 route-map Nothing-OUT out
 neighbor 33.22.21.21 remote-as 301
 neighbor 44.22.23.23 remote-as 520
!
ip prefix-list Nothing-OUT seq 5 deny 0.0.0.0/0 le 32
!
route-map Nothing-OUT permit 10
 match ip address prefix-list Nothing-OUT
!
```

```
R22#sh ip bgp
BGP table version is 21, local router ID is 10.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   0.0.0.0                  0         32768 i
 *   10.33.33.21/32   22.111.22.14                           0 1001 301 i         (!!!)
 *>                   33.22.21.21              0             0 301 i
 *   10.44.44.23/32   22.111.22.14                           0 1001 301 520 i     (!!!)
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23              0             0 520 i
 *   10.44.44.24/32   22.111.22.14                           0 1001 301 520 i     (!!!)
 *>                   44.22.23.23                            0 520 i
 *                    33.22.21.21                            0 301 520 i
 *   10.44.44.25/32   22.111.22.14                           0 1001 301 520 i     (!!!)
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.44.44.26/32   22.111.22.14                           0 1001 301 520 i     (!!!)
 *                    33.22.21.21                            0 301 520 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>                   44.22.23.23                            0 520 i
 *   10.111.3.14/32   33.22.21.21                            0 301 1001 i
 *>                   22.111.22.14             0             0 1001 i
 *>  10.111.3.15/32   22.111.22.14                           0 1001 i
 *                    33.22.21.21                            0 301 1001 i
 *   10.112.3.18/32   22.111.22.14                           0 1001 301 520 2042 i (!!!)
 *>                   33.22.21.21                            0 301 520 2042 i
 *   44.114.25.0/24   22.111.22.14                           0 1001 301 520 i      (!!!)
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.114.26.0/24   22.111.22.14                           0 1001 301 520 i      (!!!)
 *                    33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.115.25.0/24   22.111.22.14                           0 1001 301 520 i      (!!!)
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
B        10.33.33.21 [20/0] via 33.22.21.21, 04:25:27
B        10.44.44.23 [20/0] via 44.22.23.23, 04:25:17
B        10.44.44.24 [20/0] via 44.22.23.23, 04:24:46
B        10.44.44.25 [20/0] via 44.22.23.23, 04:24:46
B        10.44.44.26 [20/0] via 44.22.23.23, 04:24:46
B        10.111.3.14 [20/0] via 22.111.22.14, 00:03:27
B        10.111.3.15 [20/0] via 22.111.22.14, 00:03:27
B        10.112.3.18 [20/0] via 33.22.21.21, 04:24:25
      22.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        22.111.22.0/24 is directly connected, Ethernet1/0
L        22.111.22.22/32 is directly connected, Ethernet1/0
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        33.22.21.0/24 is directly connected, Ethernet0/1
L        33.22.21.22/32 is directly connected, Ethernet0/1
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        44.22.23.0/24 is directly connected, Ethernet0/0
L        44.22.23.22/32 is directly connected, Ethernet0/0
B        44.114.25.0/24 [20/0] via 44.22.23.23, 04:24:46
B        44.114.26.0/24 [20/0] via 44.22.23.23, 04:24:46
B        44.115.25.0/24 [20/0] via 44.22.23.23, 04:24:46
```

##### После применения фильтрации транзитного трафика на R14:

```
R22#show ip bgp
BGP table version is 23, local router ID is 10.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.22.22.22/32   0.0.0.0                  0         32768 i
 *>  10.33.33.21/32   33.22.21.21              0             0 301 i
 *   10.44.44.23/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23              0             0 520 i
 *>  10.44.44.24/32   44.22.23.23                            0 520 i
 *                    33.22.21.21                            0 301 520 i
 *   10.44.44.25/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.44.44.26/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *>  10.111.3.14/32   33.22.21.21                            0 301 1001 i
 *                    22.111.22.14             0             0 1001 1001 1001 i
 *   10.111.3.15/32   22.111.22.14                           0 1001 1001 1001 i
 *>                   33.22.21.21                            0 301 1001 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.112.3.18/32   33.22.21.21                            0 301 520 2042 i
 *   44.114.25.0/24   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.114.26.0/24   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.115.25.0/24   33.22.21.21                            0 301 520 i
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
B        10.33.33.21 [20/0] via 33.22.21.21, 04:28:19
B        10.44.44.23 [20/0] via 44.22.23.23, 04:28:09
B        10.44.44.24 [20/0] via 44.22.23.23, 04:27:38
B        10.44.44.25 [20/0] via 44.22.23.23, 04:27:38
B        10.44.44.26 [20/0] via 44.22.23.23, 04:27:38
B        10.111.3.14 [20/0] via 33.22.21.21, 00:00:51
B        10.111.3.15 [20/0] via 33.22.21.21, 00:00:51
B        10.112.3.18 [20/0] via 33.22.21.21, 04:27:17
      22.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        22.111.22.0/24 is directly connected, Ethernet1/0
L        22.111.22.22/32 is directly connected, Ethernet1/0
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        33.22.21.0/24 is directly connected, Ethernet0/1
L        33.22.21.22/32 is directly connected, Ethernet0/1
      44.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        44.22.23.0/24 is directly connected, Ethernet0/0
L        44.22.23.22/32 is directly connected, Ethernet0/0
B        44.114.25.0/24 [20/0] via 44.22.23.23, 04:27:38
B        44.114.26.0/24 [20/0] via 44.22.23.23, 04:27:38
B        44.115.25.0/24 [20/0] via 44.22.23.23, 04:27:38
```

##### После включения default route и применения фильтрации исходящих маршрутов на R22:

```
R22#show ip bgp neighbors 22.111.22.14 advertised-routes
BGP table version is 16, local router ID is 10.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

Originating default network 0.0.0.0                                         (!!!)

     Network          Next Hop            Metric LocPrf Weight Path

Total number of prefixes 0 
R22#
R22#show ip bgp                                         
BGP table version is 16, local router ID is 10.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
     0.0.0.0          0.0.0.0                                0 i            (!!!)
 *>  10.22.22.22/32   0.0.0.0                  0         32768 i
 *   10.33.33.21/32   44.22.23.23                            0 520 301 i
 *>                   33.22.21.21              0             0 301 i
 *   10.44.44.23/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23              0             0 520 i
 *   10.44.44.24/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.44.44.25/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.44.44.26/32   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   10.111.3.14/32   44.22.23.23                            0 520 301 1001 i
 *>                   33.22.21.21                            0 301 1001 i
 *                    22.111.22.14             0             0 1001 1001 1001 i
 *   10.111.3.15/32   44.22.23.23                            0 520 301 1001 i
 *                    22.111.22.14                           0 1001 1001 1001 i
 *>                   33.22.21.21                            0 301 1001 i
 *>  10.112.3.18/32   44.22.23.23                            0 520 2042 i
 *                    33.22.21.21                            0 301 520 2042 i
 *   44.114.25.0/24   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.114.26.0/24   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
 *   44.115.25.0/24   33.22.21.21                            0 301 520 i
 *>                   44.22.23.23                            0 520 i
```
