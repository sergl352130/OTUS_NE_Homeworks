# Лабораторная работа 01
+ ## Настройка маршрутизации между VLAN по методу "Router-on-a-Stick"
## **Топология** 
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/VLAN_topology.png?raw=true)

## Цели:
+ ### Часть 1: Создание сети и настройка основных параметров устройств
+ ### Часть 2: Создание VLAN и включение в них портов коммутаторов
+ ### Часть 3: Настройка транков 802.1Q между коммутаторами
+ ### Часть 4: Настройка связности между VLAN на маршрутизаторе
+ ### Часть 5: Проверка функционирования маршрутизации между VLAN

## Решение
## Часть 1: Создание сети и настройка основных параметров устройств

### Шаг 1: Коммутация сети по схеме
### Шаг 2: Настройка основных параметров маршрутизатора

#### R1:

```
R1#sh run
Building configuration...
!
service password-encryption
!
hostname R1
!
!
enable secret 5 $1$Reuv$qavlhXlH94cg/KUaY8wmq/
!
!
no ip domain lookup
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 094F471A1A0A
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 110A1016141D
 login
 transport input none
!
!
end
```

### Шаг 3: Настройка основных параметров для каждого коммутатора

#### S1:

```
S1#sh ru
Building configuration...
!
service password-encryption
!
hostname S1
!
!
enable secret 5 $1$z/jU$GTlXmNNBfZ71r2uqROprS1
!
!
no ip domain-lookup
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 070C285F4D06
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 05080F1C2243
 login
!
!
end
```

#### S2:

```
S2#sh ru
Building configuration...
!
service password-encryption
!
hostname S2
!
!
enable secret 5 $1$Rjlr$koDY7LTz5WrwYIo51EF1a/
!
!
no ip domain-lookup
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 045802150C2E
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 094F471A1A0A
 login
!
!
end
```

### Шаг 4: Настройка основных параметров для каждого ПК

#### PC-A:

```
Welcome to Virtual PC Simulator, version 1.3 (0.8.1)
Dedicated to Daling.
Build time: May  7 2022 15:27:29
Copyright (c) 2007-2015, Paul Meng (mirnshi@gmail.com)
Copyright (c) 2021, Alain Degreffe (alain.degreffe@eve-ng.net)
All rights reserved.

VPCS is free software, distributed under the terms of the "BSD" licence.
Source code and license can be found at vpcs.sf.net.
For more information, please visit wiki.freecode.com.cn.
Modified version for EVE-NG.

Press '?' to get help.

Executing the startup file


Checking for duplicate address...
PC-A : 192.168.3.3 255.255.255.0 gateway 192.168.3.1
```

#### PC-B:

```
Welcome to Virtual PC Simulator, version 1.3 (0.8.1)
Dedicated to Daling.
Build time: May  7 2022 15:27:29
Copyright (c) 2007-2015, Paul Meng (mirnshi@gmail.com)
Copyright (c) 2021, Alain Degreffe (alain.degreffe@eve-ng.net)
All rights reserved.

VPCS is free software, distributed under the terms of the "BSD" licence.
Source code and license can be found at vpcs.sf.net.
For more information, please visit wiki.freecode.com.cn.
Modified version for EVE-NG.

Press '?' to get help.

Executing the startup file


Checking for duplicate address...
PC-B : 192.168.4.3 255.255.255.0 gateway 192.168.4.1
```

## Часть 2: Создание VLAN и включение в них портов коммутаторов

### Шаг 1: Создание VLAN на обоих коммутаторах
### Шаг 2: Включение в VLANы портов коммутаторов

#### S1:

```
S1#sh ru
Building configuration...
!
interface Ethernet0/0
 switchport access vlan 3
 switchport mode access
!
!
interface Ethernet0/2
 switchport access vlan 7
 switchport mode access
 shutdown
!
!
end

S1#sh vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    Management                       active    Et0/0
4    Operations                       active
7    ParkingLot                       active    Et0/2
8    Native                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

#### S2:

```
S2#sh ru
Building configuration...
!
interface Ethernet0/0
 switchport access vlan 4
 switchport mode access
!
!
interface Ethernet0/2
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Ethernet0/3
 switchport access vlan 7
 switchport mode access
 shutdown
!
end

S2#sh vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    Management                       active
4    Operations                       active    Et0/0
7    ParkingLot                       active    Et0/2, Et0/3
8    Native                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```


## Часть 3: Настройка транков 802.1Q между коммутаторами


#### S1:

```
S1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      8
Et0/3       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Et0/1       3-4,8
Et0/3       3-4,8

Port        Vlans allowed and active in management domain
Et0/1       3-4,8
Et0/3       3-4,8

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       3-4,8
Et0/3       3-4,8
```

#### S2:

```
S2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Et0/1       3-4,8

Port        Vlans allowed and active in management domain
Et0/1       3-4,8

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       3-4,8
```


## Часть 4: Настройка связности между VLAN на маршрутизаторе


#### R1:

```
R1#sh run
Building configuration...
!
interface Ethernet0/0
 no ip address
 shutdown
!
interface Ethernet0/1
 no ip address
 shutdown
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
!
interface Ethernet0/3.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface Ethernet0/3.4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface Ethernet0/3.8
 encapsulation dot1Q 8 native
!
!
end

R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  administratively down down
Ethernet0/1                unassigned      YES NVRAM  administratively down down
Ethernet0/2                unassigned      YES NVRAM  administratively down down
Ethernet0/3                unassigned      YES NVRAM  up                    up  
Ethernet0/3.3              192.168.3.1     YES NVRAM  up                    up  
Ethernet0/3.4              192.168.4.1     YES NVRAM  up                    up  
Ethernet0/3.8              unassigned      YES unset  up                    up  
```


## Часть 5: Проверка функционирования маршрутизации между VLAN

#### PC-A:

```
PC-A> ping 192.168.3.1

84 bytes from 192.168.3.1 icmp_seq=1 ttl=255 time=0.461 ms
84 bytes from 192.168.3.1 icmp_seq=2 ttl=255 time=0.861 ms
84 bytes from 192.168.3.1 icmp_seq=3 ttl=255 time=0.757 ms
84 bytes from 192.168.3.1 icmp_seq=4 ttl=255 time=1.010 ms
84 bytes from 192.168.3.1 icmp_seq=5 ttl=255 time=0.869 ms

PC-A> ping 192.168.4.3

84 bytes from 192.168.4.3 icmp_seq=1 ttl=63 time=2.568 ms
84 bytes from 192.168.4.3 icmp_seq=2 ttl=63 time=1.644 ms
84 bytes from 192.168.4.3 icmp_seq=3 ttl=63 time=1.800 ms
84 bytes from 192.168.4.3 icmp_seq=4 ttl=63 time=1.642 ms
84 bytes from 192.168.4.3 icmp_seq=5 ttl=63 time=1.968 ms

PC-A> ping 192.168.3.12

84 bytes from 192.168.3.12 icmp_seq=1 ttl=255 time=0.490 ms
84 bytes from 192.168.3.12 icmp_seq=2 ttl=255 time=1.203 ms
84 bytes from 192.168.3.12 icmp_seq=3 ttl=255 time=0.980 ms
84 bytes from 192.168.3.12 icmp_seq=4 ttl=255 time=0.979 ms
84 bytes from 192.168.3.12 icmp_seq=5 ttl=255 time=0.568 ms
```

#### PC-B:

```
PC-B> ping 192.168.3.3

84 bytes from 192.168.3.3 icmp_seq=1 ttl=63 time=2.465 ms
84 bytes from 192.168.3.3 icmp_seq=2 ttl=63 time=1.063 ms
84 bytes from 192.168.3.3 icmp_seq=3 ttl=63 time=1.926 ms
84 bytes from 192.168.3.3 icmp_seq=4 ttl=63 time=1.178 ms
84 bytes from 192.168.3.3 icmp_seq=5 ttl=63 time=1.218 ms

PC-B> trace 192.168.3.3
trace to 192.168.3.3, 8 hops max, press Ctrl+C to stop
 1   192.168.4.1   1.253 ms  0.694 ms  0.766 ms
 2   *192.168.3.3   1.078 ms (ICMP type:3, code:3, Destination port unreachable)
```
