# Проектная работа
+ ## Модернизация магистральной сети WAN на основе технологий MPLS/SR
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/FinalProject/WAN-SR-MPBGP_topology.png?raw=true)

## Цели:
+ ### Часть 1: Изучение особенностей построения транспортных сетей MPLS на основе технологии SR
+ ### Часть 2: Оценка плюсов и минусов связки технологий MPLS/SR
+ ### Часть 3: Получение практических навыков создания сервисов L3VPN (L2VPN) в перспективных транспортных сетях провайдерского уровня

### Этапы:
+ #### Шаг 1: Разработка и документирование адресного пространства для лабораторного стенда

##### 1.1. Нумерация оборудования:

+ 01 - Москва, объект 1, client edge маршрутизатор 1 (BGP)
+ 02 - Москва, объект 2, client edge маршрутизатор 2 (BGP)
+ 11 - Москва, объект 1, provider edge маршрутизатор 1 (MPBGP/MPLS/SR/ISIS)
+ 12 - Москва, объект 2, provider edge маршрутизатор 2 (MPBGP/MPLS/SR/ISIS)
+ 21 - Москва, объект 1, provider маршрутизатор 1 (MPLS/SR/ISIS)
+ 22 - Москва, объект 2, provider маршрутизатор 2 (MPLS/SR/ISIS)
+ 31 - Нижний Новгород, объект 1, provider маршрутизатор 1 (MPLS/SR/ISIS)
+ 32 - Нижний Новгород, объект 2, provider маршрутизатор 2 (MPLS/SR/ISIS)
+ 41 - Нижний Новгород, объект 1, provider edge маршрутизатор 1 (MPBGP/MPLS/SR/ISIS)
+ 42 - Нижний Новгород, объект 2, provider edge маршрутизатор 2 (MPBGP/MPLS/SR/ISIS)
+ 03 - Нижний Новгород, объект 1, client edge маршрутизатор 1 (BGP)
+ 04 - Нижний Новгород, объект 2, client edge маршрутизатор 2 (BGP)

##### 1.2. Таблица IP-адресации:

|Device|Interface|IP Address   |Subnet Mask    |Description   |
|:----:|:--------|:------------|:--------------|:------------:|
|Москва|
|VPC-Msk-pay  |eth0     |10.0.1.2     |255.255.255.0  |                |
|VPC-Msk-info |eth0     |10.0.2.2     |255.255.255.0  |                |
|R-Msk-CE-pay |Loopback0|10.1.1.1     |255.255.255.255|                |
|R-Msk-CE-pay |E0/0     |10.0.1.1     |255.255.255.0  |To VPC-Msk-pay  |
|R-Msk-CE-pay |E0/2	|10.12.1.2    |255.255.255.252|To CSR-Msk-PE2  |
|R-Msk-CE-pay |E0/3     |10.11.1.2    |255.255.255.252|To CSR-Msk-PE1  |
|R-Msk-CE-info|Loopback0|10.2.2.1     |255.255.255.255|                |
|R-Msk-CE-info|E0/0     |10.0.2.1     |255.255.255.0  |To VPC-Msk-info |
|R-Msk-CE-info|E0/2	|10.11.2.2    |255.255.255.252|To CSR-Msk-PE1  |
|R-Msk-CE-info|E0/3	|10.12.2.2    |255.255.255.252|To CSR-Msk-PE2  |
|CSR-Msk-PE1  |Loopback0|10.11.11.1   |255.255.255.255|                |
|CSR-Msk-PE1  |Gi1      |10.21.11.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-Msk-PE1  |Gi2	|10.11.12.1   |255.255.255.252|To CSR-Msk-PE2  |
|CSR-Msk-PE1  |Gi3	|10.11.1.1    |255.255.255.252|To R-Msk-CE-pay |
|CSR-Msk-PE1  |Gi4	|10.11.2.1    |255.255.255.252|To R-Msk-CE-info|
|CSR-Msk-PE2  |Loopback0|10.12.12.1   |255.255.255.255|                |
|CSR-Msk-PE2  |Gi1      |10.22.12.2   |255.255.255.252|To CSR-Msk-P2   |
|CSR-Msk-PE2  |Gi2	|10.11.12.2   |255.255.255.252|To CSR-Msk-PE1  |
|CSR-Msk-PE2  |Gi3	|10.12.2.1    |255.255.255.252|To R-Msk-CE-info|
|CSR-Msk-PE2  |Gi4	|10.12.1.1    |255.255.255.252|To R-Msk-CE-pay |
|CSR-Msk-P1   |Loopback0|10.21.21.1   |255.255.255.255|                |
|CSR-Msk-P1   |Gi1      |10.21.11.1   |255.255.255.252|To CSR-Msk-PE1  |
|CSR-Msk-P1   |Gi2	|10.21.22.1   |255.255.255.252|To CSR-Msk-P2   |
|CSR-Msk-P1   |Gi3	|10.21.31.1   |255.255.255.252|To CSR-NN-P1    |
|CSR-Msk-P1   |Gi4	|10.11.32.1   |255.255.255.252|To CSR-NN-P2    |
|CSR-Msk-P2   |Loopback0|10.22.22.1   |255.255.255.255|                |
|CSR-Msk-P2   |Gi1      |10.22.12.1   |255.255.255.252|To CSR-Msk-PE2  |
|CSR-Msk-P2   |Gi2	|10.21.22.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-Msk-P2   |Gi3	|10.22.32.1   |255.255.255.252|To CSR-NN-P2    |
|CSR-Msk-P2   |Gi4	|10.22.31.1   |255.255.255.252|To CSR-NN-P1    |
|Нижний Новгород|
|CSR-NN-P1    |Loopback0|10.31.31.1   |255.255.255.255|                |
|CSR-NN-P1    |Gi1      |10.31.41.1   |255.255.255.252|To CSR-NN-PE1   |
|CSR-NN-P1    |Gi2	|10.31.32.1   |255.255.255.252|To CSR-NN-P2    |
|CSR-NN-P1    |Gi3	|10.21.31.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-NN-P1    |Gi4	|10.22.31.2   |255.255.255.252|To CSR-Msk-P2   |
|CSR-NN-P2    |Loopback0|10.32.32.1   |255.255.255.255|                |
|CSR-NN-P2    |Gi1      |10.32.42.1   |255.255.255.252|To CSR-NN-PE2   |
|CSR-NN-P2    |Gi2	|10.31.32.2   |255.255.255.252|To CSR-NN-P1    |
|CSR-NN-P2    |Gi3	|10.22.32.2   |255.255.255.252|To CSR-Msk-P2   |
|CSR-NN-P2    |Gi4	|10.21.32.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-NN-PE1   |Loopback0|10.41.41.1   |255.255.255.255|                |
|CSR-NN-PE1   |Gi1      |10.31.41.2   |255.255.255.252|To CSR-NN-P1    |
|CSR-NN-PE1   |Gi2	|10.41.42.1   |255.255.255.252|To CSR-NN-PE2   |
|CSR-NN-PE1   |Gi3	|10.41.3.1    |255.255.255.252|To R-NN-CE-pay  |
|CSR-NN-PE1   |Gi4	|10.41.4.1    |255.255.255.252|To R-NN-CE-info |
|CSR-NN-PE2   |Loopback0|10.42.42.1   |255.255.255.255|                |
|CSR-NN-PE2   |Gi1      |10.32.42.2   |255.255.255.252|To CSR-NN-P2    |
|CSR-NN-PE2   |Gi2	|10.41.42.2   |255.255.255.252|To CSR-NN-PE1   |
|CSR-NN-PE2   |Gi3	|10.42.4.1    |255.255.255.252|To R-NN-CE-info |
|CSR-NN-PE2   |Gi4	|10.42.3.1    |255.255.255.252|To R-NN-CE-pay  |
|R-NN-CE-pay  |Loopback0|10.3.3.1     |255.255.255.255|                |
|R-NN-CE-pay  |E0/0     |10.0.3.1     |255.255.255.0  |To VPC-NN-pay   |
|R-NN-CE-pay  |E0/2	|10.42.3.2    |255.255.255.252|To CSR-NN-PE2   |
|R-NN-CE-pay  |E0/3	|10.41.3.2    |255.255.255.252|To CSR-NN-PE1   |
|R-NN-CE-info |Loopback0|10.4.4.1     |255.255.255.255|                |
|R-NN-CE-info |E0/0     |10.0.4.1     |255.255.255.0  |To VPC-NN-info  |
|R-NN-CE-info |E0/2	|10.41.4.2    |255.255.255.252|To CSR-NN-PE1   |
|R-NN-CE-info |E0/3	|10.42.4.2    |255.255.255.252|To CSR-NN-PE2   |
|VPC-NN-pay   |eth0     |10.0.3.2     |255.255.255.0  |                |
|VPC-NN-info  |eth0     |10.0.4.2     |255.255.255.0  |                |


+ #### Шаг 2: Создание лабораторного стенда

##### 2.1. Для client edge маршрутизаторов выбраны IOL-образы Cisco "L3-ADVENTERPRISEK9-M-15.4-2T".

##### 2.2. Для MPLS/SR-маршрутизаторов выбраны QEMU-образы "csr1000vng-universalk9.17.03.03.Amsterdam" облачных маршрутизаторов Cisco CSR1000v с подходящей функциональностью. Образы доступны для загрузки по [ссылке](https://labhub.eu.org/UNETLAB%20II/addons/qemu/Cisco%20CSR1000v%2017.x). 

##### 2.3. Следует иметь ввиду, что для каждого узла CSR1000v требуется 4 Гб оперативной памяти. Имеющихся у меня 30 Гб хватает под стенд с трудом, активно используется файл подкачки. Все ноды сразу стартовать не рекомендуется, 2/3 узлов при этом "зависает". Нужно запускать каждую ноду последовательно с контролем через консоль.

##### 2.4. Ещё одна сложность - возможно связанная с тем, что у меня процессор AMD (Ryzen 7 4800H), возможно дело и не в этом - ноды с образами CSR1000v по умолчанию создаются с виртуальными сетевыми интерфейсами (QEMU Nic) "tpl(vmxnet3)". 

<img src="https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/FinalProject/QEMU_Nic1.png" width="400" height="500">

##### У меня с этими QEMU Nic протокол IP не захотел запускаться, между нодами даже элементарный ping не ходил. Решение - сразу после создания каждой ноды необходимо зайти в меню редактирования настроек ноды и поменять QEMU Nic на "virtio-net-pci". С такими сетевыми интерфейсами всё работает корректно. В чем проблема - неизвестно, скорее всего дело в сбое драйвера виртуальной сетевой карты.

<img src="https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/FinalProject/QEMU_Nic2.png" width="400" height="500">

+ #### Шаг 3: Настройка IP-адреса на каждом активном порту

+ #### Шаг 4: Настройка протокола ISIS на каждом из MPLS/SR-маршрутизаторов и обеспечение full IP connectivity в SR-домене

##### CSR-Msk-PE1:

```
!
version 17.3
!
hostname CSR-Msk-PE1
!
interface Loopback0
 no shutdown
 ip address 10.12.12.1 255.255.255.255
 ip router isis 111
!
interface GigabitEthernet1
 no shutdown
 description To CSR-Msk-P2
 ip address 10.22.12.2 255.255.255.252
 ip router isis 111
!
interface GigabitEthernet2
 no shutdown
 description To CSR-Msk2-PE1
 ip address 10.11.12.2 255.255.255.252
 ip router isis 111
!
interface GigabitEthernet3
 no shutdown
 description To R-Msk-CE-info
 ip address 10.12.2.1 255.255.255.252
!
interface GigabitEthernet4
 no shutdown
 description To R-Msk-CE-pay
 ip address 10.12.1.1 255.255.255.252
!
router isis 111
net 49.0111.0000.0000.0011.00
```

```
CSR-Msk-PE1#show isis topology 

Tag 111:

IS-IS TID 0 paths to level-2 routers
System Id            Metric     Next-Hop             Interface   SNPA
CSR-Msk-PE1          --
CSR-Msk-PE2          10         CSR-Msk-PE2          Gi2         5000.0004.0001 
CSR-Msk-P1           10         CSR-Msk-P1           Gi1         5000.0006.0000 
CSR-Msk-P2           20         CSR-Msk-PE2          Gi2         5000.0004.0001 
                                CSR-Msk-P1           Gi1         5000.0006.0000 
CSR-NN-P1            20         CSR-Msk-P1           Gi1         5000.0006.0000 
CSR-NN-P2            20         CSR-Msk-P1           Gi1         5000.0006.0000 
CSR-NN-PE1           30         CSR-Msk-P1           Gi1         5000.0006.0000 
CSR-NN-PE2           30         CSR-Msk-P1           Gi1         5000.0006.0000 
CSR-Msk-PE1#
CSR-Msk-PE1#show ip route 

      10.0.0.0/8 is variably subnetted, 22 subnets, 2 masks
C        10.11.11.1/32 is directly connected, Loopback0
C        10.11.12.0/30 is directly connected, GigabitEthernet2
L        10.11.12.1/32 is directly connected, GigabitEthernet2
i L2     10.12.12.1/32 [115/20] via 10.11.12.2, 01:26:52, GigabitEthernet2
C        10.21.11.0/30 is directly connected, GigabitEthernet1
L        10.21.11.2/32 is directly connected, GigabitEthernet1
i L2     10.21.21.1/32 [115/20] via 10.21.11.1, 01:26:41, GigabitEthernet1
i L2     10.21.22.0/30 [115/20] via 10.21.11.1, 01:26:41, GigabitEthernet1
i L2     10.21.31.0/30 [115/20] via 10.21.11.1, 01:26:41, GigabitEthernet1
i L2     10.21.32.0/30 [115/20] via 10.21.11.1, 01:26:41, GigabitEthernet1
i L2     10.22.12.0/30 [115/20] via 10.11.12.2, 01:26:52, GigabitEthernet2
i L2     10.22.22.1/32 [115/30] via 10.21.11.1, 01:26:14, GigabitEthernet1
                       [115/30] via 10.11.12.2, 01:26:14, GigabitEthernet2
i L2     10.22.31.0/30 [115/30] via 10.21.11.1, 01:26:14, GigabitEthernet1
                       [115/30] via 10.11.12.2, 01:26:14, GigabitEthernet2
i L2     10.22.32.0/30 [115/30] via 10.21.11.1, 01:26:14, GigabitEthernet1
                       [115/30] via 10.11.12.2, 01:26:14, GigabitEthernet2
i L2     10.31.31.1/32 [115/30] via 10.21.11.1, 01:26:14, GigabitEthernet1
i L2     10.31.32.0/30 [115/30] via 10.21.11.1, 01:26:14, GigabitEthernet1
i L2     10.31.41.0/30 [115/30] via 10.21.11.1, 01:26:14, GigabitEthernet1
i L2     10.32.32.1/32 [115/30] via 10.21.11.1, 01:26:19, GigabitEthernet1
i L2     10.32.42.0/30 [115/30] via 10.21.11.1, 01:26:19, GigabitEthernet1
i L2     10.41.41.1/32 [115/40] via 10.21.11.1, 01:26:14, GigabitEthernet1
i L2     10.41.42.0/30 [115/40] via 10.21.11.1, 01:26:14, GigabitEthernet1
i L2     10.42.42.1/32 [115/40] via 10.21.11.1, 01:26:19, GigabitEthernet1
CSR-Msk-PE1#
```

+ #### Шаг 4: Настройка MPLS и Segment Routing в связке с протоколом ISIS

##### 4.1. Руководства по настройке MPLS и SR

##### 1. [Segment Routing (общая информация)](https://www.segment-routing.net/)
##### 2. [Segment Routing Configuration Guide, Cisco IOS XE 17 | Access and Edge Routers](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/seg_routing/configuration/xe-17/segrt-xe-17-book.html)
##### 3. [Сети для самых маленьких. Часть десятая. Базовый MPLS](https://habr.com/ru/articles/246425/)
##### 4. [Segment routing: как и почему](https://habr.com/ru/articles/317158/)

##### 4.2. Настройка MPLS и SR

##### CSR-Msk-PE1:

```
!
segment-routing mpls
 !
 connected-prefix-sid-map
  address-family ipv4
   10.11.11.1/32 index 11 range 1 
  exit-address-family
 !
 router isis 111
 net 49.0111.0000.0000.0011.00
 is-type level-2-only
 metric-style wide
 segment-routing mpls
 !
 ```