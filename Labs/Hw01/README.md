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

### R1:

```
R1#sh run
Building configuration...
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

### S1:

```
S1#sh ru
Building configuration...
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

### S2:

```
S2#sh ru
Building configuration...
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


## Часть 2: Создание VLAN и включение в них портов коммутаторов

### Шаг 1: Создание VLAN на обоих коммутаторах
### Шаг 2: Включение в VLANы портов коммутаторов

### S1:

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

### S2:

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


### **S1**:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/S1(tr).png?raw=true)


### **S2**:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/S2(tr).png?raw=true)





## **Часть 4: Configure Inter-VLAN Routing on the Router**


a.	Activate interface G0/0/1 on the router.


b.	Configure sub-interfaces for each VLAN as specified in the IP addressing table. All sub-interfaces use 802.1Q encapsulation. Ensure the sub-interface for the native VLAN does not have an IP address assigned. Include a description for each sub-interface.


c.	Use the show ip interface brief command to verify the sub-interfaces are operational.


### **R1**:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/RoT.png?raw=true)


## **Часть 5: Verify Inter-VLAN Routing is Working**

### **Шаг 1: Complete the following tests from PC-A. All should be successful.**



a.	Ping from PC-A to its default gateway.


b.	Ping from PC-A to PC-B


c.	Ping from PC-A to S2


### **PC-A**:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/PC-A%20Test.png?raw=true)



### **Шаг 2: Complete the following test from PC-B.**

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/Tracert.png?raw=true)



