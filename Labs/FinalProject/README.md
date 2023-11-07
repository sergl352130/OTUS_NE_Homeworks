# Проектная работа
+ ## Модернизация магистральной сети WAN на основе технологий MPLS/SR
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/FinalProject/WAN-SR-MPBGP_topology.png?raw=true)

## Цели:
+ ### Часть 1: Изучение особенностей построения транспортных сетей MPLS на основе технологии SR
+ ### Часть 2: Оценка плюсов и минусов связки технологий MPLS/SR
+ ### Часть 3: Получение практических навыков создания сервисов L3VPN (L2VPN) в перспективных транспортных сетях провайдерского уровня

### Этапы:
+ #### Шаг 1: Разработка и документирование адресного пространства для лабораторного стенда

##### Принята следующая схема формирования IP-адресов:
###### 1. Нумерация оборудования:
+ 01 Москва, объект 1, BGP-маршрутизатор СE1
+ 02 Москва, объект 2, BGP-маршрутизатор СE2
+ 11 Москва, объект 1, MPLS/SR-маршрутизатор PE1
+ 12 Москва, объект 2, MPLS/SR-маршрутизатор PE2
+ 21 Москва, объект 1, MPLS/SR-маршрутизатор P1
+ 22 Москва, объект 2, MPLS/SR-маршрутизатор P2
+ 31 Нижний Новгород, объект 1, MPLS/SR-маршрутизатор P1
+ 32 Нижний Новгород, объект 2, MPLS/SR-маршрутизатор P2
+ 41 Нижний Новгород, объект 1, MPLS/SR-маршрутизатор PE1
+ 42 Нижний Новгород, объект 2, MPLS/SR-маршрутизатор PE2
+ 03 Нижний Новгород, объект 1, BGP-маршрутизатор СE1
+ 04 Нижний Новгород, объект 2, BGP-маршрутизатор СE2


###### 2. Для связей провайдер/провайдер - №провайдера1.№провайдера2.№роутера1.№роутера2. Пример - 44.22.23.22, Триада.Киторн.R23(Триада).R22(Киторн). На другом конце - 44.22.23.23, Триада.Киторн.R23(Триада).R23(Триада).
###### 3. Для связей провайдер/клиент - №провайдера.№клиента.№роутераРЕ.№роутераСЕ. Пример - 33.111.21.15, Ламас.Москва.R21(Ламас).R15(Москва). На другом конце - 33.111.21.21, Ламас.Москва.R21(Ламас).R21(Ламас).
###### 4. Для локальных сетей офисов/провайдеров выбраны частные адреса из диапазона 10.0.0.0/8 и p2p-подсети с маской /30.


|Device|Interface|IP Address   |Subnet Mask    |Description   |
|:----:|:--------|:------------|:--------------|:------------:|
|Москва|
|VPC1  |eth0     |DHCP	       |DHCP	         |DHCP          |
|VPC7  |eth0     |DHCP	       |DHCP	         |DHCP          |
|SW2   |VLAN111  |10.111.3.102 |255.255.255.240|VLAN111 Mgmt  |
|SW3   |VLAN111  |10.111.3.103 |255.255.255.240|VLAN111 Mgmt  |
|SW4   |Loopback0|10.111.3.4   |255.255.255.255|SW4 Mgmt      |
|SW4   |VLAN111  |10.111.3.104 |255.255.255.240|VLAN111 Mgmt  |
|SW4   |E1/0	   |10.111.2.1   |255.255.255.252|p2p to R12    |
|SW4   |VLAN11	 |10.111.0.1   |255.255.255.0  |DefGW VLAN11  |
|SW5   |Loopback0|10.111.3.5   |255.255.255.255|SW5 Mgmt      |
|SW5   |VLAN111  |10.111.3.105 |255.255.255.240|VLAN111 Mgmt  |
|SW5   |E1/0	   |10.111.2.5   |255.255.255.252|p2p to R13    |
|SW5   |VLAN17	 |10.111.1.1   |255.255.255.0  |DefGWy VLAN17 |
|R12   |Loopback0|10.111.3.12  |255.255.255.255|R12 Mgmt      |
|R12   |E1/0     |10.111.2.2   |255.255.255.252|p2p to SW4    |
|R12   |E0/0     |10.111.2.9   |255.255.255.252|p2p to R14    |
|R12   |E0/1     |10.111.2.13  |255.255.255.252|p2p to R15    |
|R13   |Loopback0|10.111.3.13  |255.255.255.255|R13 Mgmt      |
|R13   |E1/0     |10.111.2.6   |255.255.255.252|p2p to SW5    |
|R13   |E0/0     |10.111.2.17  |255.255.255.252|p2p to R15    |
|R13   |E0/1     |10.111.2.21  |255.255.255.252|p2p to R14    |
|R14   |Loopback0|10.111.3.14  |255.255.255.255|R14 Mgmt      |
|R14   |E0/0     |10.111.2.10  |255.255.255.252|p2p to R12    |
|R14   |E0/1     |10.111.2.22  |255.255.255.252|p2p to R13    |
|R14   |E0/2     |10.111.2.25  |255.255.255.252|p2p to R15    |
|R14   |E0/3     |10.111.2.29  |255.255.255.252|p2p to R19    |
|R14   |E1/0     |22.111.22.14 |255.255.255.0  |to R22 Kitorn |
|R15   |Loopback0|10.111.3.15  |255.255.255.255|R15 Mgmt      |
|R15   |E0/0     |10.111.2.18  |255.255.255.252|p2p to R13    |
|R15   |E0/1     |10.111.2.14  |255.255.255.252|p2p to R12    |
|R15   |E0/2     |10.111.2.26  |255.255.255.252|p2p to R14    |
|R15   |E0/3     |10.111.2.33  |255.255.255.252|p2p to R20    |
|R15   |E1/0     |33.111.21.15 |255.255.255.0  |to R21 Lamas  |
|R19   |Loopback0|10.111.3.19  |255.255.255.255|R19 Mgmt      |
|R19   |E0/3     |10.111.2.30  |255.255.255.252|p2p to R14    |
|R20   |Loopback0|10.111.3.20  |255.255.255.255|R20 Mgmt      |
|R20   |E0/3     |10.111.2.34  |255.255.255.252|p2p to R15    |
|С.-Петербург|
|VPC8  |eth0     |10.112.0.2   |255.255.255.0  |VPC8          |
|VPC   |eth0     |10.112.1.2   |255.255.255.0  |VPC           |
|SW9   |Loopback0|10.112.3.9   |255.255.255.255|SW9 Mgmt      |
|SW9   |E1/0	   |10.112.2.1   |255.255.255.252|p2p to R17    |
|SW9   |VLAN18	 |10.112.0.1   |255.255.255.0  |DefGW VLAN18  |
|SW10  |Loopback0|10.112.3.10  |255.255.255.255|SW10 Mgmt     |
|SW10  |E1/0	   |10.112.2.9   |255.255.255.252|p2p to R16    |
|SW10  |VLAN10	 |10.112.1.1   |255.255.255.0  |DefGW VLAN10  |
|R17   |Loopback0|10.112.3.17  |255.255.255.255|R17 Mgmt      |
|R17   |E1/0     |10.112.2.2   |255.255.255.252|p2p to SW9    |
|R17   |E0/0     |10.112.2.17  |255.255.255.252|p2p to R18    |
|R17   |E0/2     |10.112.2.5   |255.255.255.252|p2p to R16    |
|R16   |Loopback0|10.112.3.16  |255.255.255.255|R16 Mgmt      |
|R16   |E1/0     |10.112.2.10  |255.255.255.252|p2p to SW10   |
|R16   |E0/1     |10.112.2.21  |255.255.255.252|p2p to R18    |
|R16   |E0/2     |10.112.2.6   |255.255.255.252|p2p to R17    |
|R16   |E0/3     |10.112.2.25  |255.255.255.252|p2p to R32    |
|R18   |Loopback0|10.112.3.18  |255.255.255.255|R18 Mgmt      |
|R18   |E0/0     |10.112.2.18  |255.255.255.252|p2p to R17    |
|R18   |E0/1     |10.112.2.22  |255.255.255.252|p2p to R16    |
|R18   |E1/0     |44.112.24.18 |255.255.255.0  |to R24 Triada |
|R18   |E1/1     |44.112.26.18 |255.255.255.0  |to R26 Triada |
|R32   |Loopback0|10.112.3.32  |255.255.255.255|R32 Mgmt      |
|R32   |E0/3     |10.112.2.26  |255.255.255.252|p2p to R16    |
|Чокурдах|
|VPC30 |eth0     |10.111.4.66  |255.255.255.192|VPC30         |
|VPC31 |eth0     |10.111.4.131 |255.255.255.192|VPC31         |
|SW29  |Loopback0|10.111.4.29  |255.255.255.255|SW29 Mgmt     |
|R28   |Loopback0|10.111.4.28  |255.255.255.255|R28 Mgmt      |
|R28   |E0/0.30  |10.111.4.65  |255.255.255.192|LAN VPC30     |
|R28   |E0/0.31  |10.111.4.129 |255.255.255.192|LAN VPC31     |
|R28   |E1/0     |44.114.26.28 |255.255.255.0  |to R26 Triada |
|R28   |E1/1     |44.114.25.28 |255.255.255.0  |to R25 Triada |
|Лабытнанги|
|R27   |Loopback0|10.111.5.27  |255.255.255.255|R27 Mgmt      |
|R27   |E0/0     |10.111.5.129 |255.255.255.128|internal LAN  |
|R27   |E1/0     |44.115.25.27 |255.255.255.0  |to R25 Triada |
|Киторн|
|R22   |Loopback0|10.22.22.22  |255.255.255.255|R22 Mgmt      |
|R22   |E0/0     |44.22.23.22  |255.255.255.0  |to R23 Triada |
|R22   |E0/1     |33.22.21.22  |255.255.255.0  |to R21 Lamas  |
|R22   |E1/0     |22.111.22.22 |255.255.255.0  |to R14 Moscow |
|Ламас |
|R21   |Loopback0|10.33.33.21  |255.255.255.255|R21 Mgmt      |
|R21   |E0/0     |44.33.24.24  |255.255.255.0  |to R24 Triada |
|R21   |E0/1     |33.22.21.21  |255.255.255.0  |to R22 Kitorn |
|R21   |E1/0     |33.111.21.21 |255.255.255.0  |to R15 Moscow |
|Триада|
|R23   |Loopback0|10.44.44.23  |255.255.255.255|R23 Mgmt      |
|R23   |E0/0     |44.22.23.23  |255.255.255.0  |to R22 Kitorn |
|R23   |E0/1     |10.44.44.1   |255.255.255.252|p2p to R25    |
|R23   |E0/2     |10.44.44.5   |255.255.255.252|p2p to R24    |
|R24   |Loopback0|10.44.44.24  |255.255.255.255|R24 Mgmt      |
|R24   |E0/0     |44.33.24.24  |255.255.255.0  |to R21 Lamas  |
|R24   |E0/1     |10.44.44.9   |255.255.255.252|p2p to R26    |
|R24   |E0/2     |10.44.44.6   |255.255.255.252|p2p to R23    |
|R24   |E1/0     |44.112.24.24 |255.255.255.0  |to R18 SPb    |
|R25   |Loopback0|10.44.44.25  |255.255.255.255|R25 Mgmt      |
|R25   |E0/1     |10.44.44.2   |255.255.255.252|p2p to R23    |
|R25   |E0/2     |10.44.44.13  |255.255.255.252|p2p to R26    |
|R25   |E1/0     |44.115.25.25 |255.255.255.0  |to R27 Lab-gi |
|R25   |E1/1     |44.114.25.25 |255.255.255.0  |to R28 Chok   |
|R26   |Loopback0|10.44.44.26  |255.255.255.255|R26 Mgmt      |
|R26   |E0/1     |10.44.44.10  |255.255.255.252|p2p to R24    |
|R26   |E0/2     |10.44.44.14  |255.255.255.252|p2p to R25    |
|R26   |E1/0     |44.114.26.26 |255.255.255.0  |to R28 Chok   |
|R26   |E1/1     |44.112.26.26 |255.255.255.0  |to R18 SPb    |


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