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

###### 2. Таблица IP-адресации:

|Device|Interface|IP Address   |Subnet Mask    |Description   |
|:----:|:--------|:------------|:--------------|:------------:|
|Москва|
|VPC-Msk-pay  |eth0     |10.0.1.2     |255.255.255.0  |                |
|VPC-Msk-info |eth0     |10.0.2.2     |255.255.255.0  |                |
|R-Msk-CE-pay |Loopback0|10.1.1.1     |255.255.255.255|                |
|R-Msk-CE-pay |E0/0     |10.0.1.1     |255.255.255.0  |To VPC-Msk-pay  |
|R-Msk-CE-pay |E0/2	    |10.12.1.2    |255.255.255.252|To CSR-Msk-PE2  |
|R-Msk-CE-pay |E0/3	    |10.11.1.2    |255.255.255.252|To CSR-Msk-PE1  |
|R-Msk-CE-info|Loopback0|10.2.2.1     |255.255.255.255|                |
|R-Msk-CE-info|E0/0     |10.0.2.1     |255.255.255.0  |To VPC-Msk-info |
|R-Msk-CE-info|E0/2	    |10.11.2.2    |255.255.255.252|To CSR-Msk-PE1  |
|R-Msk-CE-info|E0/3	    |10.12.2.2    |255.255.255.252|To CSR-Msk-PE2  |
|CSR-Msk-PE1  |Loopback0|10.11.11.1   |255.255.255.255|                |
|CSR-Msk-PE1  |Gi1      |10.21.11.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-Msk-PE1  |Gi2	    |10.11.12.1   |255.255.255.252|To CSR-Msk-PE2  |
|CSR-Msk-PE1  |Gi3	    |10.11.1.1    |255.255.255.252|To R-Msk-CE-pay |
|CSR-Msk-PE1  |Gi4	    |10.11.2.1    |255.255.255.252|To R-Msk-CE-info|
|CSR-Msk-PE2  |Loopback0|10.12.12.1   |255.255.255.255|                |
|CSR-Msk-PE2  |Gi1      |10.22.12.2   |255.255.255.252|To CSR-Msk-P2   |
|CSR-Msk-PE2  |Gi2	    |10.11.12.2   |255.255.255.252|To CSR-Msk-PE1  |
|CSR-Msk-PE2  |Gi3	    |10.12.2.1    |255.255.255.252|To R-Msk-CE-info|
|CSR-Msk-PE2  |Gi4	    |10.12.1.1    |255.255.255.252|To R-Msk-CE-pay |
|CSR-Msk-P1   |Loopback0|10.21.21.1   |255.255.255.255|                |
|CSR-Msk-P1   |Gi1      |10.21.11.1   |255.255.255.252|To CSR-Msk-PE1  |
|CSR-Msk-P1   |Gi2	    |10.21.22.1   |255.255.255.252|To CSR-Msk-P2   |
|CSR-Msk-P1   |Gi3	    |10.21.31.1   |255.255.255.252|To CSR-NN-P1    |
|CSR-Msk-P1   |Gi4	    |10.11.32.1   |255.255.255.252|To CSR-NN-P2    |
|CSR-Msk-P2   |Loopback0|10.22.22.1   |255.255.255.255|                |
|CSR-Msk-P2   |Gi1      |10.22.12.1   |255.255.255.252|To CSR-Msk-PE2  |
|CSR-Msk-P2   |Gi2	    |10.21.22.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-Msk-P2   |Gi3	    |10.22.32.1   |255.255.255.252|To CSR-NN-P2    |
|CSR-Msk-P2   |Gi4	    |10.22.31.1   |255.255.255.252|To CSR-NN-P1    |
|Нижний Новгород|
|CSR-NN-P1    |Loopback0|10.31.31.1   |255.255.255.255|                |
|CSR-NN-P1    |Gi1      |10.31.41.1   |255.255.255.252|To CSR-NN-PE1   |
|CSR-NN-P1    |Gi2	    |10.31.32.1   |255.255.255.252|To CSR-NN-P2    |
|CSR-NN-P1    |Gi3	    |10.21.31.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-NN-P1    |Gi4	    |10.22.31.2   |255.255.255.252|To CSR-Msk-P2   |
|CSR-NN-P2    |Loopback0|10.32.32.1   |255.255.255.255|                |
|CSR-NN-P2    |Gi1      |10.32.42.1   |255.255.255.252|To CSR-NN-PE2   |
|CSR-NN-P2    |Gi2	    |10.31.32.2   |255.255.255.252|To CSR-NN-P1    |
|CSR-NN-P2    |Gi3	    |10.22.32.2   |255.255.255.252|To CSR-Msk-P2   |
|CSR-NN-P2    |Gi4	    |10.21.32.2   |255.255.255.252|To CSR-Msk-P1   |
|CSR-NN-PE1   |Loopback0|10.41.41.1   |255.255.255.255|                |
|CSR-NN-PE1   |Gi1      |10.31.41.2   |255.255.255.252|To CSR-NN-P1    |
|CSR-NN-PE1   |Gi2	    |10.41.42.1   |255.255.255.252|To CSR-NN-PE2   |
|CSR-NN-PE1   |Gi3	    |10.41.3.1    |255.255.255.252|To R-NN-CE-pay  |
|CSR-NN-PE1   |Gi4	    |10.41.4.1    |255.255.255.252|To R-NN-CE-info |
|CSR-NN-PE2   |Loopback0|10.42.42.1   |255.255.255.255|                |
|CSR-NN-PE2   |Gi1      |10.32.42.2   |255.255.255.252|To CSR-NN-P2    |
|CSR-NN-PE2   |Gi2	    |10.41.42.2   |255.255.255.252|To CSR-NN-PE1   |
|CSR-NN-PE2   |Gi3	    |10.42.4.1    |255.255.255.252|To R-NN-CE-info |
|CSR-NN-PE2   |Gi4	    |10.42.3.1    |255.255.255.252|To R-NN-CE-pay  |
|R-NN-CE-pay  |Loopback0|10.3.3.1     |255.255.255.255|                |
|R-NN-CE-pay  |E0/0     |10.0.3.1     |255.255.255.0  |To VPC-NN-pay   |
|R-NN-CE-pay  |E0/2	    |10.42.3.2    |255.255.255.252|To CSR-NN-PE2   |
|R-NN-CE-pay  |E0/3	    |10.41.3.2    |255.255.255.252|To CSR-NN-PE1   |
|R-NN-CE-info |Loopback0|10.4.4.1     |255.255.255.255|                |
|R-NN-CE-info |E0/0     |10.0.4.1     |255.255.255.0  |To VPC-NN-info  |
|R-NN-CE-info |E0/2	    |10.41.4.2    |255.255.255.252|To CSR-NN-PE1   |
|R-NN-CE-info |E0/3	    |10.42.4.2    |255.255.255.252|To CSR-NN-PE2   |
|VPC-NN-pay   |eth0     |10.0.3.2     |255.255.255.0  |                |
|VPC-NN-info  |eth0     |10.0.4.2     |255.255.255.0  |                |


+ #### Шаг 2: Настройка IP-адреса на каждом активном порту

  + Примеры конфигураций:

#### SW4:

```

```