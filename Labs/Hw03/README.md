# Лабораторная работа 03
+ ## Настройка и внедрение DHCPv4/v6
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw03/DHCP_topology.png?raw=true)

## Цели (DHCPv4):
+ ### Часть 1: Создание сети и настройка основных параметров устройств
+ ### Часть 2: Настройка и верификация двух DHCPv4 серверов на маршрутизаторе R1
+ ### Часть 3: Настройка и верификация ретрансляции DHCP на маршрутизаторе R2

## Цели (DHCPv6):
+ ### Часть 1: Создание сети и настройка основных параметров устройств
+ ### Часть 2: Верификация присвоения адреса по протоколу SLAAC с маршрутизатора R1
+ ### Часть 3: Настройка и верификация Stateless DHCPv6 сервера на маршрутизаторе R1
+ ### Часть 4: Настройка и верификация Stateful DHCPv6 сервера на маршрутизаторе R1
+ ### Часть 5: Настройка и верификация ретрансляции DHCP на маршрутизаторе R2


# Решение (DHCPv4)

## Часть 1: Создание сети и настройка основных параметров устройств

### Шаг 1: Разработка схемы адресации

#### Разделить сеть 192.168.1.0/24 в соответствии со следующими требованиями:
+ ##### Подсеть А поддерживает 58 хостов (клиентский VLAN на маршрутизаторе R1).
Подсеть А: 192.168.1.0/26
+ ##### Подсеть В поддерживает 28 хостов (VLAN управления на маршрутизаторе R1).
Подсеть В: 192.168.1.64/27
+ ##### Подсеть С поддерживает 12 хостов (клиентский VLAN на маршрутизаторе R2).
Подсеть В: 192.168.1.96/28

|Device|Interface|IP Address  |Subnet Mask    |Default Gateway|
|:----:|:--------|:-----------|:--------------|:-------------:|
|R1	   |E0/0	   |10.0.0.1    |255.255.255.252|N/A            |
|R1	   |E0/1	   |N/A	        |N/A	          |N/A            |
|R1	   |E0/1.100 |192.168.1.1 |255.255.255.192|N/A            |
|R1	   |E0/1.200 |192.168.1.65|255.255.255.224|N/A            |
|R1	   |E0/1.1000|N/A         |N/A	          |N/A            |
|R2	   |E0/0	   |10.0.0.2    |255.255.255.252|N/A            |
|R2	   |E0/1	   |192.168.1.97|255.255.255.240|N/A            |
|S1	   |VLAN 200 |192.168.1.66|255.255.255.224|192.168.1.65   |
|S2	   |VLAN 1	 |192.168.1.98|255.255.255.240|192.168.1.97   |
|PC-A  |NIC	     |DHCP	      |DHCP	          |DHCP           |
|PC-B  |NIC	     |DHCP	      |DHCP	          |DHCP           |

### Шаг 2: Коммутация сети по схеме
### Шаг 3: Настройка основных параметров для каждого маршрутизатора

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
enable secret 5 $1$uVzv$yXPXUYM28Zo/oyQYdHe1q0
!
!
clock timezone MSK 3 0
!
!
no ip domain lookup
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
 password 7 094F471A1A0A
 login
 transport input none
!
!
end
```

#### R2:

```
R2#sh run
Building configuration...
!
service password-encryption
!
hostname R2
!
!
enable secret 5 $1$Gc3c$f2SmFX0gmzwRtcaaez/fJ0
!
!
clock timezone MSK 3 0
!
!
no ip domain lookup
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 14141B180F0B
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 030752180500
 login
 transport input none
!
!
end
```

### Шаг 4: Настройка связности между VLAN на маршрутизаторе R1

#### R1:

```
R1#sh run
Building configuration...
!
!
interface Ethernet0/0
 description "to R2"
 ip address 10.0.0.1 255.255.255.252
!
interface Ethernet0/1
 description "to Clients"
 no ip address
!
interface Ethernet0/1.100
 description "Clients VLAN"
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
!
interface Ethernet0/1.200
 description "Management VLAN"
 encapsulation dot1Q 200
 ip address 192.168.1.65 255.255.255.224
!
interface Ethernet0/1.1000
 description "Native VLAN"
 encapsulation dot1Q 1000 native
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
!
ip route 0.0.0.0 0.0.0.0 10.0.0.2
!
!
end

R1#show interfaces ethernet 0/1 summary

 *: interface is up
 IHQ: pkts in input hold queue     IQD: pkts dropped from input queue
 OHQ: pkts in output hold queue    OQD: pkts dropped from output queue
 RXBS: rx rate (bits/sec)          RXPS: rx rate (pkts/sec)
 TXBS: tx rate (bits/sec)          TXPS: tx rate (pkts/sec)
 TRTL: throttle count

  Interface                   IHQ       IQD       OHQ       OQD      RXBS      RXPS      TXBS      TXPS      TRTL
-----------------------------------------------------------------------------------------------------------------
* Ethernet0/1                   0         0         0         0         0         1         0         0         0
* Ethernet0/1.100               -         -         -         -         -         -         -         -         -
* Ethernet0/1.200               -         -         -         -         -         -         -         -         -
* Ethernet0/1.1000              -         -         -         -         -         -         -         -         -
NOTE:No separate counters are maintained for subinterfaces
     Hence Details of subinterface are not shown

R1#show interfaces ethernet 0/1.100
Ethernet0/1.100 is up, line protocol is up
  Hardware is AmdP2, address is aabb.cc00.1010 (bia aabb.cc00.1010)
  Description: "Clients VLAN"
  Internet address is 192.168.1.1/26
  MTU 1500 bytes, BW 10000 Kbit/sec, DLY 1000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID  100.
  ARP type: ARPA, ARP Timeout 04:00:00
  Keepalive set (10 sec)
  Last clearing of "show interface" counters never

R1#show interfaces ethernet 0/1.200
Ethernet0/1.200 is up, line protocol is up
  Hardware is AmdP2, address is aabb.cc00.1010 (bia aabb.cc00.1010)
  Description: "Management VLAN"
  Internet address is 192.168.1.65/27
  MTU 1500 bytes, BW 10000 Kbit/sec, DLY 1000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID  200.
  ARP type: ARPA, ARP Timeout 04:00:00
  Keepalive set (10 sec)
  Last clearing of "show interface" counters never

R1#show interfaces ethernet 0/1.1000
Ethernet0/1.1000 is up, line protocol is up
  Hardware is AmdP2, address is aabb.cc00.1010 (bia aabb.cc00.1010)
  Description: "Native VLAN"
  MTU 1500 bytes, BW 10000 Kbit/sec, DLY 1000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID  1000.
  ARP type: ARPA, ARP Timeout 04:00:00
  Keepalive set (10 sec)
  Last clearing of "show interface" counters never
```

### Шаг 5: Настройка интерфейсов на R2 и статической маршрутизации для обоих маршрутизаторов

#### R2:

```
R2#sh run
Building configuration...
!
!
interface Ethernet0/0
 description "to R1"
 ip address 10.0.0.2 255.255.255.252
!
interface Ethernet0/1
 description "to Clients"
 ip address 192.168.1.97 255.255.255.240
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
!
ip route 0.0.0.0 0.0.0.0 10.0.0.1
!
!
end

R1#ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### Шаг 6: Настройка основных параметров для каждого коммутатора

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
enable secret 5 $1$HMS5$BIAyrbU/WT/oVC5WF5FpS.
!
!
clock timezone MSK 3 0
!
!
no ip domain-lookup
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 060506324F41
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 030752180500
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
enable secret 5 $1$Stj7$gaWsRDNd/6DUNkxmDiqH4/
!
clock timezone MSK 3 0
!
no ip domain-lookup
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 0822455D0A16
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

### Шаг 7: Создание VLAN на коммутаторе S1 (S2 остается с базовыми настройками)

### Шаг 8: Присвоение VLAN соответствующим интерфейсам на обоих коммутаторах

#### S1:

```
S1#sh run
Building configuration...
!
interface Ethernet0/0
 switchport access vlan 999
 switchport mode access
 shutdown
!
interface Ethernet0/1
!
interface Ethernet0/2
 switchport access vlan 999
 switchport mode access
 shutdown
!
interface Ethernet0/3
 switchport access vlan 100
 switchport mode access
!
interface Vlan200
 ip address 192.168.1.66 255.255.255.224
!
ip default-gateway 192.168.1.65
!
!
end

S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0
100  Clients                          active    Et0/3
200  Management                       active
999  Parking_Lot                      active    Et0/0, Et0/2
1000 Native                           active
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
!
interface Ethernet0/1
!
interface Ethernet0/2
!
interface Ethernet0/3
!
interface Vlan1
 ip address 192.168.1.98 255.255.255.240
!
ip default-gateway 192.168.1.97
!
!
end

S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

### Ответ на вопрос:

  1. #### Почему интерфейс Е0/1 выводится в качестве присвоенного VLAN1?
     + Это происходит потому, что интерфейс Е0/1 на данном шаге не был настроен ни в качестве транкового, ни в качестве порта доступа.

### Шаг 9: На коммутаторе S1 настроить интерфейс Е0/1 в качестве транкового

#### S1:

```
S1#sh run
Building configuration...
!
interface Ethernet0/1
 switchport trunk allowed vlan 100,200,1000
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 1000
 switchport mode trunk
!
!
end

S1#sh int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      1000

Port        Vlans allowed on trunk
Et0/1       100,200,1000

Port        Vlans allowed and active in management domain
Et0/1       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       100,200,1000
```

### Ответ на вопрос:

  2. #### На этом шаге какой IP-адрес получат компьютеры при подключении к сети постредством протокола DHCP?
     + **PC-A** 169.254.107.237/16; **PC-B** 169.254.218.79/16


## Часть 2: Настройка и верификация двух DHCPv4 серверов на маршрутизаторе R1

### Шаг 1: Настроить на маршрутизаторе R1 пулы DHCPv4 для двух подсетей
### Шаг 2: Сохранить конфигурацию
### Шаг 3: Проверить конфигурацию сервера DHCPv4

#### R1:

```
R1#sh run
Building configuration...
!
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.97 192.168.1.101
!
ip dhcp pool SubnetA
 network 192.168.1.0 255.255.255.192
 domain-name otus-lab.ru
 default-router 192.168.1.1
 lease 2 12 30
!
ip dhcp pool SubnetC
 network 192.168.1.96 255.255.255.240
 domain-name otus-lab.ru
 default-router 192.168.1.97
 lease 2 12 30
!
!
end

R1#show ip dhcp pool

Pool SubnetA :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 62
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.7          192.168.1.1      - 192.168.1.62      1

Pool SubnetC :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 14
 Leased addresses               : 0
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.97         192.168.1.97     - 192.168.1.110     0

R1#show ip dhcp binding

Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.6         0150.0000.0700.00       May 20 2023 02:34 PM    Automatic

R1#show ip dhcp server statistics

Memory usage         33622
Address pools        2
Database agents      0
Automatic bindings   1
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         8
DHCPREQUEST          1
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0

Message              Sent
BOOTREPLY            0
DHCPOFFER            1
DHCPACK              1
DHCPNAK              0
```

### Шаг 4: Инициализировать на РС-А попытку получения IP адреса по протоколу DHCP

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw03/PC-A_DHCPv4.png?raw=true)

## Часть 3: Настройка и верификация ретрансляции DHCP на маршрутизаторе R2

### Шаг 1: Настроить на маршрутизаторе R2 агента ретрансляции DHCP

#### R2:

```
R2#sh run
Building configuration...
!
interface Ethernet0/1
 description "to Clients"
 ip address 192.168.1.97 255.255.255.240
 ip helper-address 10.0.0.1
!
!
end
```

### Шаг 2: Инициализировать на РС-B попытку получения IP адреса по протоколу DHCP

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw03/PC-B_DHCPv4.png?raw=true)

#### R1:

```
R1#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.6         0150.0000.0700.00       May 22 2023 03:30 AM    Automatic
192.168.1.102       0150.0000.0800.00       May 22 2023 03:48 AM    Automatic

R1#show ip dhcp server statistics
Memory usage         42081
Address pools        2
Database agents      0
Automatic bindings   2
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         9
DHCPREQUEST          5
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           5

Message              Sent
BOOTREPLY            0
DHCPOFFER            2
DHCPACK              10
DHCPNAK              0
```


# Решение (DHCPv6)

## Часть 1: Создание сети и настройка основных параметров устройств

|Device|Interface|IPv6 Address         |
|:----:|:--------|:--------------------|
|R1	 |E0/0	  |2001:db8:acad:2::1/64|
|      |E0/0     |fe80::1              |
|R1	 |E0/1.100 |2001:db8:acad:1::1/64|
|      |E0/1.100 |fe80::1              |
|R2	 |E0/0	  |2001:db8:acad:2::2/64|
|      |E0/0     |fe80::2              |
|R2	 |E0/1	  |2001:db8:acad:3::1/64|
|      |E0/1     |fe80::1              |
|PC-A  |NIC	     |DHCP	               |
|PC-B  |NIC	     |DHCP	               |

### Шаг 1: Коммутация сети по схеме
### Шаг 2: Настройка основных параметров для каждого коммутатора

#### S1 (аналогично DHCPv4)
#### S2 (аналогично DHCPv4)

### Шаг 3: Настройка основных параметров для каждого маршрутизатора
### Шаг 4: Настройка интерфейсов и маршрутизации для каждого маршрутизатора

#### R1:

```
R1#sh run
Building configuration...
!
ipv6 unicast-routing
ipv6 cef
!
interface Ethernet0/0
 description "to R2"
 ip address 10.0.0.1 255.255.255.252
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
!
interface Ethernet0/1.100
 description "Clients VLAN"
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
!
ip forward-protocol nd
!
ipv6 route ::/0 2001:DB8:ACAD:2::2
!
!
end
```

#### R2:

```
R2#sh run
Building configuration...
!
ipv6 unicast-routing
ipv6 cef
!
interface Ethernet0/0
 description "to R1"
 ip address 10.0.0.2 255.255.255.252
 ipv6 address FE80::2 link-local
 ipv6 address 2001:DB8:ACAD:2::2/64
!
interface Ethernet0/1
 description "to Clients"
 ip address 192.168.1.97 255.255.255.240
 ip helper-address 10.0.0.1
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:3::1/64
!
ip forward-protocol nd
!
ipv6 route ::/0 2001:DB8:ACAD:2::1
!
!
end
```

#### R1:

```
R1#ping 2001:db8:acad:3::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:ACAD:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```


## Часть 2: Верификация присвоения адреса по протоколу SLAAC с маршрутизатора R1

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw03/PC-A_SLAAC.png?raw=true)

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw03/PC-B_SLAAC.png?raw=true)


## Часть 3: Настройка и верификация Stateless DHCPv6 сервера на маршрутизаторе R1

### Шаг 1: Расширенная проверка конфигурации хоста PC-A

```
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\user>ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : user-PC (PC-A)
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : otus-lab.ru

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : otus-lab.ru
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 50-00-00-05-00-00
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : 2001:db8:acad:1:64a1:d610:e046:f714(Preferred)
   Temporary IPv6 Address. . . . . . : 2001:db8:acad:1:fcd4:f21b:c909:1d30(Preferred)
   Link-local IPv6 Address . . . . . : fe80::64a1:d610:e046:f714%11(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.6(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.192
   Lease Obtained. . . . . . . . . . : Thursday, May 25, 2023 3:18:30 PM
   Lease Expires . . . . . . . . . . : Sunday, May 28, 2023 3:48:30 AM
   Default Gateway . . . . . . . . . : fe80::1%11
                                       192.168.1.1
   DHCP Server . . . . . . . . . . . : 192.168.1.1
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1
                                       fec0:0:0:ffff::2%1
                                       fec0:0:0:ffff::3%1
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

### Шаг 2: Настройка stateless DHCPv6 на маршрутизаторе R1 для хоста PC-A

#### R1:

```
R1#sh run
Building configuration...
!
ipv6 unicast-routing
ipv6 cef
ipv6 dhcp pool R1-Stateless
 dns-server 2001:DB8:ACAD::254
 domain-name stateless.com
!
!
interface Ethernet0/1.100
 description "Clients VLAN"
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server R1-Stateless
!
ip forward-protocol nd
!
ipv6 route ::/0 2001:DB8:ACAD:2::2
!
!
end
```

#### PC-A:

```
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\user>ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : user-PC (PC-A)
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : stateless.com

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : otus-lab.ru
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 50-00-00-05-00-00
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : 2001:db8:acad:1:64a1:d610:e046:f714(Preferred)
   Temporary IPv6 Address. . . . . . : 2001:db8:acad:1:e134:f64e:79dd:eff0(Preferred)
   Link-local IPv6 Address . . . . . : fe80::64a1:d610:e046:f714%11(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.6(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.192
   Lease Obtained. . . . . . . . . . : Thursday, May 25, 2023 4:53:23 PM
   Lease Expires . . . . . . . . . . : Sunday, May 28, 2023 5:23:22 AM
   Default Gateway . . . . . . . . . : fe80::1%11
                                       192.168.1.1
   DHCP Server . . . . . . . . . . . : 192.168.1.1
   DHCPv6 IAID . . . . . . . . . . . : 240126464
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-21-72-34-36-50-0A-00-01-00-00
   DNS Servers . . . . . . . . . . . : 2001:db8:acad::254
   NetBIOS over Tcpip. . . . . . . . : Enabled
   Connection-specific DNS Suffix Search List :
                                       stateless.com

C:\Users\user>ping 2001:db8:acad:3::1

Pinging 2001:db8:acad:3::1 with 32 bytes of data:
Reply from 2001:db8:acad:3::1: time=1ms
Reply from 2001:db8:acad:3::1: time=1ms
Reply from 2001:db8:acad:3::1: time=1ms
Reply from 2001:db8:acad:3::1: time<1ms

Ping statistics for 2001:db8:acad:3::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

## Часть 4: Настройка и верификация Stateful DHCPv6 сервера на маршрутизаторе R1

#### R1:

```
R1#sh run
Building configuration...
!
ipv6 unicast-routing
ipv6 cef
ipv6 dhcp pool R1-Stateless
 dns-server 2001:DB8:ACAD::254
 domain-name stateless.com
!
ipv6 dhcp pool R2-Stateful
 address prefix 2001:DB8:ACAD:3:AAA::/80
 dns-server 2001:DB8:ACAD::254
 domain-name statefull.com
!
interface Ethernet0/0
 description "to R2"
 ip address 10.0.0.1 255.255.255.252
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
 ipv6 dhcp server R2-Stateful
!
interface Ethernet0/1
 description "to Clients"
 no ip address
!
interface Ethernet0/1.100
 description "Clients VLAN"
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server R1-Stateless
!
ip forward-protocol nd
!
ipv6 route ::/0 2001:DB8:ACAD:2::2
!
!
end
```

## Часть 5: Настройка и верификация ретрансляции DHCP на маршрутизаторе R2

### Шаг 1: Включение хоста PC-B и проверка выданного по протоколу SLAAC адреса

#### PC-B:

```
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\user>ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : user-PC (Win-PC-B)
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : otus-lab.ru

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : otus-lab.ru
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 50-00-00-06-00-00
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : 2001:db8:acad:3:c192:15d4:3ba5:957c(Preferred)
   Temporary IPv6 Address. . . . . . : 2001:db8:acad:3:34a8:706e:57ed:2341(Preferred)
   Link-local IPv6 Address . . . . . : fe80::c192:15d4:3ba5:957c%11(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.102(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.240
   Lease Obtained. . . . . . . . . . : Thursday, May 25, 2023 10:11:06 PM
   Lease Expires . . . . . . . . . . : Sunday, May 28, 2023 10:41:06 AM
   Default Gateway . . . . . . . . . : fe80::1%11
                                       192.168.1.97
   DHCP Server . . . . . . . . . . . : 10.0.0.1
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1
                                       fec0:0:0:ffff::2%1
                                       fec0:0:0:ffff::3%1
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

### Шаг 2: Настройка на маршрутизаторе R2 агента ретрансляции DHCPv6 для сети на интерфейсе Е0/1

#### R2:

```
R2#sh run
Building configuration...
!
ipv6 unicast-routing
ipv6 cef
!
!
interface Ethernet0/0
 description "to R1"
 ip address 10.0.0.2 255.255.255.252
 ipv6 address FE80::2 link-local
 ipv6 address 2001:DB8:ACAD:2::2/64
!
interface Ethernet0/1
 description "to Clients"
 ip address 192.168.1.97 255.255.255.240
 ip helper-address 10.0.0.1
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:3::1/64
 ipv6 nd managed-config-flag
 ipv6 dhcp relay destination 2001:DB8:ACAD:2::1 Ethernet0/0
!
!
ip forward-protocol nd
!
ipv6 route ::/0 2001:DB8:ACAD:2::1
!
!
end
```

### Шаг 3: Инициализировать на РС-B попытку получения IPv6 адреса по протоколу DHCPv6

#### PC-B:

```
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\user>ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : user-PC
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : statefull.com

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : otus-lab.ru
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 50-00-00-06-00-00
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : 2001:db8:acad:3:aaa:1bab:b96a:94b8(Preferred)
   Lease Obtained. . . . . . . . . . : Thursday, May 25, 2023 10:40:48 PM
   Lease Expires . . . . . . . . . . : Saturday, May 27, 2023 10:33:29 PM
   IPv6 Address. . . . . . . . . . . : 2001:db8:acad:3:c192:15d4:3ba5:957c(Preferred)
   Temporary IPv6 Address. . . . . . : 2001:db8:acad:3:2836:7ff6:c502:6d1c(Preferred)
   Link-local IPv6 Address . . . . . : fe80::c192:15d4:3ba5:957c%11(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.102(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.240
   Lease Obtained. . . . . . . . . . : Thursday, May 25, 2023 10:40:47 PM
   Lease Expires . . . . . . . . . . : Sunday, May 28, 2023 11:10:47 AM
   Default Gateway . . . . . . . . . : fe80::1%11
                                       192.168.1.97
   DHCP Server . . . . . . . . . . . : 10.0.0.1
   DHCPv6 IAID . . . . . . . . . . . : 240126464
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-21-72-34-36-50-0A-00-01-00-00
   DNS Servers . . . . . . . . . . . : 2001:db8:acad::254
   NetBIOS over Tcpip. . . . . . . . : Enabled
   Connection-specific DNS Suffix Search List :
                                       statefull.com

C:\Users\user>ping 2001:db8:acad:1::1

Pinging 2001:db8:acad:1::1 with 32 bytes of data:
Reply from 2001:db8:acad:1::1: time=1ms
Reply from 2001:db8:acad:1::1: time=1ms
Reply from 2001:db8:acad:1::1: time=1ms
Reply from 2001:db8:acad:1::1: time=1ms

Ping statistics for 2001:db8:acad:1::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms
```