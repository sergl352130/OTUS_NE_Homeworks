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
 neighbor 44.22.23.22 remote-as 101
!
```

```

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
 neighbor 44.33.24.21 remote-as 301
 neighbor 44.112.24.18 remote-as 2042
!
```

```

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
 neighbor 44.112.26.18 remote-as 2042
!
```

```

```