# Лабораторная работа 04
+ ## Проектирование сети
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw04/Network_topology.png?raw=true)

## Цели:
+ ### Часть 1: Планировка адресного пространства
+ ### Часть 2: Настройка IP-адресации на всех активных портах сетевых устройств для дальнейшей работы над проектом
+ ### Часть 3: Документирование адресного пространства
+ ### Часть 4: Задел на будущее - настройка DHCP в офисе Москва, удаление/добавление связей

### Этапы:
+ #### Шаг 1: Разработка и документирование адресного пространства для лабораторного стенда

|Network        |Summary net  |Description  |Segment          |
|:--------------|:------------|:-----------:|:---------------:|
|10.111.0.0/24  |10.111.0.0/22|Москва       |LAN VPC1         |
|10.111.1.0/24  |10.111.0.0/22|Москва       |LAN VPC7         |
|10.111.2.0/24  |10.111.0.0/22|Москва       |p2p networks     |
|10.111.3.0/24  |10.111.0.0/22|Москва       |loopbacks mgmt   |
|10.111.4.0/26  |10.111.4.0/24|Чокурдах     |loopbacks mgmt   |
|10.111.4.64/26 |10.111.4.0/24|Чокурдах     |LAN VPC30        |
|10.111.4.128/25|10.111.4.0/24|Чокурдах     |LAN VPC31        |
|10.111.5.0/25  |10.111.5.0/24|Лабытнанги   |loopbacks mgmt   |
|10.111.5.128/25|10.111.5.0/24|Лабытнанги   |LAN              |
|10.112.0.0/24  |10.112.0.0/22|С.-Петербург |LAN VPC8         |
|10.112.1.0/24  |10.112.0.0/22|С.-Петербург |LAN VPC          |
|10.112.2.0/24  |10.112.0.0/22|С.-Петербург |p2p networks     |
|10.112.3.0/24  |10.112.0.0/22|С.-Петербург |loopbacks mgmt   |
|10.22.22.0/24  |10.22.22.0/24|Киторн       |loopbacks mgmt   |   
|22.22.22.0/24  |22.22.22.0/24|Киторн       |to clients       |
|23.23.23.0/24  |23.23.23.0/24|Киторн-Ламас |provider2provider|
|24.24.24.0/24  |24.24.24.0/24|Киторн-Триада|provider2provider|
|10.22.22.0/24  |10.21.21.0/24|Ламас        |loopbacks mgmt   |
|33.33.33.0/24  |33.33.33.0/24|Ламас        |to clients       |
|34.34.34.0/24  |34.34.34.0/24|Ламас-Триада |provider2provider|
|10.44.44.0/24  |10.44.44.0/24|Триада       |loopbacks mgmt   |
|44.44.44.0/24  |44.44.44.0/24|Триада       |to clients       |



|Device|Interface|IP Address  |Subnet Mask    |Description    |
|:----:|:--------|:-----------|:--------------|:-------------:|
|Москва|
|VPC1  |eth0     |DHCP	      |DHCP	          |DHCP           |
|VPC7  |eth0     |DHCP	      |DHCP	          |DHCP           |
|SW2   |VLAN111  |10.111.3.102|255.255.255.240|VLAN111 Mgmt   |
|SW3   |VLAN111  |10.111.3.103|255.255.255.240|VLAN111 Mgmt   |
|SW4   |Loopback0|10.111.3.4  |255.255.255.255|SW4 Mgmt       |
|SW4   |VLAN111  |10.111.3.104|255.255.255.240|VLAN111 Mgmt   |
|SW4   |E1/0	 |10.111.2.1  |255.255.255.252|p2p to R12     |
|SW4   |Vlan11	 |10.111.0.1  |255.255.255.0  |Default gateway VLAN11|
|SW5   |Loopback0|10.111.3.5  |255.255.255.255|SW5 Mgmt       |
|SW5   |VLAN111  |10.111.3.105|255.255.255.240|VLAN111 Mgmt   |
|SW5   |E1/0	 |10.111.2.5  |255.255.255.252|p2p to R13     |
|SW5   |Vlan17	 |10.111.1.1  |255.255.255.0  |Default gateway VLAN17|
|R12   |Loopback0|10.111.3.12 |255.255.255.255|R12 Mgmt       |
|R12   |E1/0     |10.111.2.2  |255.255.255.252|p2p to SW4     |
|R12   |E0/0     |10.111.2.9  |255.255.255.252|p2p to R14     |
|R12   |E0/1     |10.111.2.13 |255.255.255.252|p2p to R15     |
|R13   |Loopback0|10.111.3.13 |255.255.255.255|R13 Mgmt       |
|R13   |E1/0     |10.111.2.6  |255.255.255.252|p2p to SW5     |
|R13   |E0/0     |10.111.2.17 |255.255.255.252|p2p to R15     |
|R13   |E0/1     |10.111.2.21 |255.255.255.252|p2p to R14     |
|R14   |Loopback0|10.111.3.14 |255.255.255.255|R14 Mgmt       |
|R14   |E0/0     |10.111.2.10 |255.255.255.252|p2p to R12     |
|R14   |E0/1     |10.111.2.22 |255.255.255.252|p2p to R13     |
|R14   |E0/2     |10.111.2.25 |255.255.255.252|p2p to R15     |
|R14   |E0/3     |10.111.2.29 |255.255.255.252|p2p to R19     |
|R14   |E1/0     |22.22.22.2  |255.255.255.252|p2p to R22 Kitorn|
|R15   |Loopback0|10.111.3.15 |255.255.255.255|R15 Mgmt       |
|R15   |E0/0     |10.111.2.18 |255.255.255.252|p2p to R13     |
|R15   |E0/1     |10.111.2.14 |255.255.255.252|p2p to R12     |
|R15   |E0/2     |10.111.2.26 |255.255.255.252|p2p to R14     |
|R15   |E0/3     |10.111.2.33 |255.255.255.252|p2p to R20     |
|R15   |E1/0     |33.33.33.2  |255.255.255.252|p2p to R21 Lamas|
|R19   |Loopback0|10.111.3.19 |255.255.255.255|R19 Mgmt       |
|R19   |E0/3     |10.111.2.30 |255.255.255.252|p2p to R14     |
|R20   |Loopback0|10.111.3.20 |255.255.255.255|R20 Mgmt       |
|R20   |E0/3     |10.111.2.34 |255.255.255.252|p2p to R15     |
|С.-Петербург|
|VPC8  |eth0     |10.112.0.2  |255.255.255.0  |VPC8           |
|VPC   |eth0     |10.112.1.2  |255.255.255.0  |VPC            |
|SW9   |Loopback0|10.112.3.9  |255.255.255.255|SW9 Mgmt       |
|SW9   |E1/0	 |10.112.2.1  |255.255.255.252|p2p to R17     |
|SW9   |Vlan18	 |10.112.0.1  |255.255.255.0  |Default gateway VLAN18|
|SW10  |Loopback0|10.112.3.10 |255.255.255.255|SW10 Mgmt      |
|SW10  |E1/0	 |10.112.2.9  |255.255.255.252|p2p to R16     |
|SW10  |Vlan10	 |10.112.1.1  |255.255.255.0  |Default gateway VLAN10|
|R17   |Loopback0|10.112.3.17 |255.255.255.255|R17 Mgmt       |
|R17   |E1/0     |10.112.2.2  |255.255.255.252|p2p to SW9     |
|R17   |E0/0     |10.112.2.17 |255.255.255.252|p2p to R18     |
|R17   |E0/2     |10.112.2.5  |255.255.255.252|p2p to R16     |
|R16   |Loopback0|10.112.3.16 |255.255.255.255|R16 Mgmt       |
|R16   |E1/0     |10.112.2.10 |255.255.255.252|p2p to SW10    |
|R16   |E0/1     |10.112.2.21 |255.255.255.252|p2p to R18     |
|R16   |E0/2     |10.112.2.6  |255.255.255.252|p2p to R17     |
|R16   |E0/3     |10.112.2.25 |255.255.255.252|p2p to R32     |
|R18   |Loopback0|10.112.3.18 |255.255.255.255|R18 Mgmt       |
|R18   |E0/0     |10.112.2.18 |255.255.255.252|p2p to R17     |
|R18   |E0/1     |10.112.2.22 |255.255.255.252|p2p to R16     |
|R18   |E1/0     |44.44.44.2  |255.255.255.252|p2p to R24 Triada|
|R18   |E1/1     |44.44.44.6  |255.255.255.252|p2p to R26 Triada|
|R32   |Loopback0|10.112.3.32 |255.255.255.255|R32 Mgmt       |
|R32   |E0/3     |10.112.2.26 |255.255.255.252|p2p to R16     |
|Чокурдах|
|VPC30 |eth0     |10.111.4.66 |255.255.255.192|VPC30          |
|VPC31 |eth0     |10.111.4.130|255.255.255.128|VPC31          |
|SW29  |VLAN111	 |10.111.4.29 |255.255.255.252|SW29 Mgmt      |
|R28   |Loopback0|10.111.4.28 |255.255.255.255|R28 Mgmt       |
|R28   |E0/0.30  |10.111.4.65 |255.255.255.192|LAN VPC30      |
|R28   |E0/0.31  |10.111.4.129|255.255.255.128|LAN VPC31      |
|R28   |E0/0.111 |10.111.4.30 |255.255.255.252|SW29 Mgmt      |
|R28   |E1/0     |44.44.44.10 |255.255.255.252|p2p to R26 Triada|
|R28   |E1/1     |44.44.44.14 |255.255.255.252|p2p to R25 Triada|
|Лабытнанги|
|R27   |Loopback0|10.111.5.27 |255.255.255.255|R27 Mgmt       |
|R27   |E0/0     |10.111.5.129|255.255.255.128|internal LAN   |
|R27   |E1/0     |44.44.44.18 |255.255.255.252|p2p to R25 Triada|
|Киторн|
|R22   |Loopback0|10.22.22.22 |255.255.255.255|R22 Mgmt       |
|R22   |E0/0     |24.24.24.1  |255.255.255.252|p2p to R23 Triada|
|R22   |E0/1     |23.23.23.1  |255.255.255.252|p2p to R21 Lamas|
|R22   |E1/0     |22.22.22.1  |255.255.255.252|p2p to R14 Moscow|
|Ламас|
|R21   |Loopback0|10.21.21.21 |255.255.255.255|R21 Mgmt       |
|R21   |E0/0     |34.34.34.1  |255.255.255.252|p2p to R24 Triada|
|R21   |E0/1     |23.23.23.2  |255.255.255.252|p2p to R22 Kitorn|
|R21   |E1/0     |33.33.33.1  |255.255.255.252|p2p to R15 Moscow|
|Триада|
|R23   |Loopback0|10.44.44.23 |255.255.255.255|R23 Mgmt       |
|R23   |E0/0     |24.24.24.2  |255.255.255.252|p2p to R22 Kitorn|
|R23   |E0/1     |10.44.44.1  |255.255.255.252|p2p to R25     |
|R23   |E0/2     |10.44.44.5  |255.255.255.252|p2p to R24     |
|R24   |Loopback0|10.44.44.24 |255.255.255.255|R24 Mgmt       |
|R24   |E0/0     |34.34.34.2  |255.255.255.252|p2p to R21 Lamas|
|R24   |E0/1     |10.44.44.9  |255.255.255.252|p2p to R26     |
|R24   |E0/2     |10.44.44.6  |255.255.255.252|p2p to R23     |
|R24   |E1/0     |44.44.44.1  |255.255.255.252|p2p to R18 SPb |
|R25   |Loopback0|10.44.44.25 |255.255.255.255|R25 Mgmt       |
|R25   |E0/1     |10.44.44.2  |255.255.255.252|p2p to R23     |
|R25   |E0/2     |10.44.44.13 |255.255.255.252|p2p to R26     |
|R25   |E1/0     |44.44.44.17 |255.255.255.252|p2p to R27 Lab-gi|
|R25   |E1/1     |44.44.44.13 |255.255.255.252|p2p to R28 Chok|
|R26   |Loopback0|10.44.44.26 |255.255.255.255|R26 Mgmt       |
|R26   |E0/1     |10.44.44.10 |255.255.255.252|p2p to R24     |
|R26   |E0/2     |10.44.44.14 |255.255.255.252|p2p to R25     |
|R26   |E1/0     |44.44.44.9  |255.255.255.252|p2p to R28 Chok|
|R26   |E1/1     |44.44.44.5  |255.255.255.252|p2p to R18 SPb |


+ #### Шаг 2: Настройка IP-адреса на каждом активном порту

  + Примеры конфигураций:

#### SW4:

```
!
interface Loopback0
 no shutdown
 description SW4 Mgmt
 ip address 10.111.3.4 255.255.255.255
!
interface Port-channel45
 no shutdown
 description to SW5
 switchport trunk allowed vlan 11,17,77,111
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 77
 switchport mode trunk
!
interface Ethernet0/0
 no shutdown
 description to SW3
 switchport trunk allowed vlan 11,77,111
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 77
 switchport mode trunk
!
interface Ethernet0/1
 no shutdown
 description to SW2
 switchport trunk allowed vlan 17,77,111
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 77
 switchport mode trunk
!
interface Ethernet0/2
 no shutdown
 switchport trunk allowed vlan 11,17,77,111
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 77
 switchport mode trunk
 channel-group 45 mode on
!
interface Ethernet0/3
 no shutdown
 switchport trunk allowed vlan 11,17,77,111
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 77
 switchport mode trunk
 channel-group 45 mode on
!
interface Ethernet1/0
 no shutdown
 description p2p to R12
 no switchport
 ip address 10.111.2.1 255.255.255.252
 duplex auto
!
interface Ethernet1/1
 no shutdown
 switchport access vlan 99
 switchport mode access
 shutdown
!
interface Ethernet1/2
 no shutdown
 switchport access vlan 99
 switchport mode access
 shutdown
!
interface Ethernet1/3
 no shutdown
 switchport access vlan 99
 switchport mode access
 shutdown
!
interface Vlan11
 no shutdown
 description Default gateway VLAN11
 ip address 10.111.0.1 255.255.255.0
 ip helper-address 10.111.2.2 
!
interface Vlan111
 no shutdown
 description VLAN111 Mgmt
 ip address 10.111.3.104 255.255.255.240
!
```

#### R14:

```
!
interface Loopback0
 no shutdown
 description R14 Mgmt
 ip address 10.111.3.14 255.255.255.255
!
interface Ethernet0/0
 no shutdown
 description p2p to R12
 ip address 10.111.2.10 255.255.255.252
!
interface Ethernet0/1
 no shutdown
 ip address 10.111.2.22 255.255.255.252
!
interface Ethernet0/2
 no shutdown
 description p2p to R15
 ip address 10.111.2.25 255.255.255.252
!
interface Ethernet0/3
 no shutdown
 description p2p to R19
 ip address 10.111.2.29 255.255.255.252
!
interface Ethernet1/0
 no shutdown
 description p2p to R22 Kitorn
 ip address 22.22.22.2 255.255.255.252
!
```

+ #### Шаг 3: Настройка отдельного VLAN для каждого VPC в каждом офисе
  
#### SW3:

```
SW3#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
11   VPC1                             active    Et0/2
77   Native                           active
99   Parking                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3
111  Mgmt                             active


VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
11   enet  100011     1500  -      -      -        -    -        0      0
77   enet  100077     1500  -      -      -        -    -        0      0
99   enet  100099     1500  -      -      -        -    -        0      0
111  enet  100111     1500  -      -      -        -    -        0      0

```

#### SW2:

```
SW2#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
17   VPC7                             active    Et0/2
77   Native                           active
99   Parking                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3
111  Mgmt                             active


VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
17   enet  100017     1500  -      -      -        -    -        0      0
77   enet  100077     1500  -      -      -        -    -        0      0
99   enet  100099     1500  -      -      -        -    -        0      0
111  enet  100111     1500  -      -      -        -    -        0      0

```

#### SW9:

```
SW9#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
18   VPC8                             active    Et0/0
88   Native                           active
99   Parking                          active    Et0/1, Et1/1, Et1/2, Et1/3
112  Mgmt                             active


VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
18   enet  100018     1500  -      -      -        -    -        0      0
88   enet  100088     1500  -      -      -        -    -        0      0
99   enet  100099     1500  -      -      -        -    -        0      0
112  enet  100112     1500  -      -      -        -    -        0      0

```

#### SW10:

```
SW10#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
10   VPC                              active    Et0/0
88   Native                           active
99   Parking                          active    Et0/1, Et1/1, Et1/2, Et1/3
112  Mgmt                             active


VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
88   enet  100088     1500  -      -      -        -    -        0      0
99   enet  100099     1500  -      -      -        -    -        0      0
112  enet  100112     1500  -      -      -        -    -        0      0

```

#### SW29:

```
SW29#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
30   VPC30                            active    Et0/1
31   VPC31                            active    Et0/2
77   Native                           active
99   Parking                          active    Et0/3
111  Mgmt                             active


VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
30   enet  100030     1500  -      -      -        -    -        0      0
31   enet  100031     1500  -      -      -        -    -        0      0
77   enet  100077     1500  -      -      -        -    -        0      0
99   enet  100099     1500  -      -      -        -    -        0      0
111  enet  100111     1500  -      -      -        -    -        0      0

```


+ #### Шаг 4: Настройка интерфейсов VLAN/Loopback для управления сетевыми устройствами
  + Примеры конфигураций в п.2.
+ #### Шаг 5: Настройка сетей офисов с целью предотвращения broadcast штормов и оптимального использования линков
  
  
#### SW4:

```
SW4#show spanning-tree

VLAN0011
  Spanning tree enabled protocol rstp
  Root ID    Priority    24587
             Address     aabb.cc00.4000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24587  (priority 24576 sys-id-ext 11)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Po45                Desg FWD 56        128.65   Shr

VLAN0017
  Spanning tree enabled protocol rstp
  Root ID    Priority    24593
             Address     aabb.cc00.5000
             Cost        56
             Port        65 (Port-channel45)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32785  (priority 32768 sys-id-ext 17)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Po45                Root FWD 56        128.65   Shr

VLAN0077
  Spanning tree enabled protocol rstp
  Root ID    Priority    24653
             Address     aabb.cc00.4000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24653  (priority 24576 sys-id-ext 77)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Po45                Desg FWD 56        128.65   Shr

VLAN0111
  Spanning tree enabled protocol rstp
  Root ID    Priority    24687
             Address     aabb.cc00.5000
             Cost        56
             Port        65 (Port-channel45)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32879  (priority 32768 sys-id-ext 111)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Po45                Root FWD 56        128.65   Shr

```

#### SW5:

```
SW5#show spanning-tree

VLAN0011
  Spanning tree enabled protocol rstp
  Root ID    Priority    24587
             Address     aabb.cc00.4000
             Cost        56
             Port        65 (Port-channel45)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32779  (priority 32768 sys-id-ext 11)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Po45                Root FWD 56        128.65   Shr

VLAN0017
  Spanning tree enabled protocol rstp
  Root ID    Priority    24593
             Address     aabb.cc00.5000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24593  (priority 24576 sys-id-ext 17)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Po45                Desg FWD 56        128.65   Shr

VLAN0077
  Spanning tree enabled protocol rstp
  Root ID    Priority    24653
             Address     aabb.cc00.4000
             Cost        56
             Port        65 (Port-channel45)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32845  (priority 32768 sys-id-ext 77)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Po45                Root FWD 56        128.65   Shr

VLAN0111
  Spanning tree enabled protocol rstp
  Root ID    Priority    24687
             Address     aabb.cc00.5000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24687  (priority 24576 sys-id-ext 111)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Po45                Desg FWD 56        128.65   Shr

```
#### SW3:

```
SW3#show spanning-tree

VLAN0011
  Spanning tree enabled protocol rstp
  Root ID    Priority    24587
             Address     aabb.cc00.4000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32779  (priority 32768 sys-id-ext 11)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr

VLAN0077
  Spanning tree enabled protocol rstp
  Root ID    Priority    24653
             Address     aabb.cc00.4000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32845  (priority 32768 sys-id-ext 77)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr

VLAN0111
  Spanning tree enabled protocol rstp
  Root ID    Priority    24687
             Address     aabb.cc00.5000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32879  (priority 32768 sys-id-ext 111)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr

```
#### SW2:

```
SW2#show spanning-tree

VLAN0017
  Spanning tree enabled protocol rstp
  Root ID    Priority    24593
             Address     aabb.cc00.5000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32785  (priority 32768 sys-id-ext 17)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr

VLAN0077
  Spanning tree enabled protocol rstp
  Root ID    Priority    24653
             Address     aabb.cc00.4000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32845  (priority 32768 sys-id-ext 77)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr

VLAN0111
  Spanning tree enabled protocol rstp
  Root ID    Priority    24687
             Address     aabb.cc00.5000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32879  (priority 32768 sys-id-ext 111)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr

```
+ #### Шаг 6: Настройка IPv4. IPv6 по желанию
  + Выполнено в части IPv4. Конфигурации элементов сети в прилагаемом архиве лабораторной работы.


+ #### Шаг 7: Задел на будущее - настройка DHCP в офисе Москва

#### R12:

```
!
version 15.4
!
hostname R12
!
!
ip dhcp excluded-address 10.111.0.1 10.111.0.5
!
ip dhcp pool VPC1
 network 10.111.0.0 255.255.255.0
 default-router 10.111.0.1
 domain-name VPC1
 lease infinite
!
```

#### SW4:

```
!
version 15.2
!
hostname SW4
!
!
interface Vlan11
 no shutdown
 description Default gateway VLAN11
 ip address 10.111.0.1 255.255.255.0
 ip helper-address 10.111.2.2 
!
```

#### R13:

```
!
version 15.4
!
hostname R13
!
!
ip dhcp excluded-address 10.111.1.1 10.111.1.5
!
ip dhcp pool VPC7
 network 10.111.1.0 255.255.255.0
 default-router 10.111.1.1
 domain-name VPC7
 lease infinite
!
```

#### SW5:

```
!
version 15.2
!
hostname SW5
!
!
interface Vlan17
 description Default gateway VLAN17
 ip address 10.111.1.1 255.255.255.0
 ip helper-address 10.111.2.6
!
```

+ #### Шаг 8: Задел на будущее - удаление/добавление архитектурных связей для дальнейших лабораторных работ

Учитывая, что выбрана архитектура с L3-коммутаторами SW4/SW5 (Москва) и и L2-домен, соответственно, заканчивается на этих коммутаторах, в начальную схему лабораторной работы внесены следующие изменения:

##### Москва

+ Удалены связи SW4 - R13 и SW5 - R12, так как с точки зрения маршрутизации и резервирования они избыточны, а порты маршрутизаторов целесообразно экономить.
+ Добавлена связь R14 - R15 для обеспечения целостности Area 0 OSPF и связи по iBGP по кратчайшему пути.
+ HSRP/VRRP не настраивались, так как при такой схеме распределния L2/L3-доменов необходимость в протоколах резервирования первого перехода отсутствует

##### С.-Петербург

+ Cвязи SW9 - R16 и SW10 - R17 для оптимизации маршрутов заменены прямым линком R16 - R17 с меньшим количеством хопов.
+ DHCP и HSRP/VRRP не настраивались ввиду их избыточности для данного офиса.