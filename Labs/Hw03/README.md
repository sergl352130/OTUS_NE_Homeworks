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