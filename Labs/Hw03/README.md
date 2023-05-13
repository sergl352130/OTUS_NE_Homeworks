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

Subnet the network 192.168.1.0/24 to meet the following requirements:
a.	One subnet, “Subnet A”, supporting 58 hosts (the client VLAN at R1).
Subnet A:
Type your answers here.
Record the first IP address in the Addressing Table for R1 G0/0/1.100. 

b.	One subnet, “Subnet B”, supporting 28 hosts (the management VLAN at R1). 
Subnet B:
Type your answers here.
Record the first IP address in the Addressing Table for R1 G0/0/1.200. Record the second IP address in the Address Table for S1 VLAN 200 and enter the associated default gateway.

c.	One subnet, “Subnet C”, supporting 12 hosts (the client network at R2).
Subnet C:
Type your answers here.
Record the first IP address in the Addressing Table for R2 G0/0/1.

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
hostname R1
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