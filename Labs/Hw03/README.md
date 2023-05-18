# Лабораторная работа 03
+ ## Настройка и внедрение DHCP v4/v6
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw03/DHCP_topology.png?raw=true)

## Цели:
+ ### Часть 1: Создание сети и настройка основных параметров устройств
+ ### Часть 2: Настройка и верификация двух DHCPv4 серверов на маршрутизаторе R1
+ ### Часть 3: Настройка и верификация ретрансляции DHCP на маршрутизаторе R2

## Решение
## Часть 1: Создание сети и настройка основных параметров устройств

### Шаг 1: Разработка схемы адресации

#### Разделить сеть 192.168.1.0/24 в соответствии со следующими требованиями:
+ ##### Подсеть А поддерживает 58 хостов (клиентский VLAN на маршрутизаторе R1).
Подсеть А: 192.168.1.1/26
+ ##### Подсеть В поддерживает 28 хостов (VLAN управления на маршрутизаторе R1).
Подсеть В: 192.168.1.64/27
+ ##### Подсеть С поддерживает 12 хостов (клиентский VLAN на маршрутизаторе R2).
Подсеть В: 192.168.1.96/28

|Device|Interface|IP Address  |Subnet Mask    |Default Gateway|
|:----:|:--------|:-----------|:--------------|:-------------:|
|R1	   |E0/0	 |10.0.0.1    |255.255.255.252|N/A            |
|R1	   |E0/1	 |N/A	      |N/A	          |N/A            |
|R1	   |E0/1.100 |192.168.1.1 |255.255.255.192|N/A            |
|R1	   |E0/1.200 |192.168.1.65|255.255.255.224|N/A            |
|R1	   |E0/1.1000|N/A         |N/A	          |N/A            |
|R2	   |E0/0	 |10.0.0.2    |255.255.255.252|N/A            |
|R2	   |E0/1	 |192.168.1.97|255.255.255.240|N/A            |
|S1	   |VLAN 200 |192.168.1.66|255.255.255.224|192.168.1.65   |
|S2	   |VLAN 1	 |	          |	              |               |
|PC-A  |NIC	     |DHCP	      |DHCP	          |DHCP           |
|PC-B  |NIC	     |DHCP	      |DHCP	          |DHCP           |

### Шаг 2: Коммутация сети по схеме
### Шаг 3: Настройка основных параметров для каждого маршрутизатора

### R1:

```
R1#sh run
Building configuration...
!
hostname R1
!
!
enable secret 5 $1$uVzv$yXPXUYM28Zo/oyQYdHe1q0
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

### R2:

```
R1#sh run
Building configuration...
!
hostname R2
!
!
enable secret 5 $1$Gc3c$f2SmFX0gmzwRtcaaez/fJ0
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

### Шаг 5: Настройка интерфейсов на R2 и статической маршрутизации для обоих маршрутизаторов

### Шаг 6: Настройка основных параметров для каждого коммутатора

#### S1:

```
S1#sh ru
Building configuration...
!
hostname S1
!
!
enable secret 5 $1$HMS5$BIAyrbU/WT/oVC5WF5FpS.
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
hostname S2
!
!
enable secret 5 $1$Stj7$gaWsRDNd/6DUNkxmDiqH4/
!
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

S1#sh vlan brief

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

### Ответ на вопрос:

  #### Почему интерфейс Е0/1 выводится в качестве присвоенного VLAN1?
     + Это происходит потому, что интерфейс Е0/1 на данном шаге не был настроен ни в качестве транкового, ни в качестве порта доступа.

### Шаг 9: На коммутаторе S1 настроить интерфейс Е0/1 в качестве транкового

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

  #### At this point, what IP address would the PC’s have if they were connected to the network using DHCP?
     + х.
