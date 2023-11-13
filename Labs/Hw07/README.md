# Лабораторная работа 07
+ ## ISIS
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw06/Network_topology_OSPF.png?raw=true)

## Цели:
+ ### Настройка IS-IS в офисе Триада

### Этапы:
+ #### Шаг 1: Настройка IS-IS в ISP Триада
+ #### Шаг 2: R23 и R25 находятся в зоне 2222
+ #### Шаг 3: R24 находится в зоне 24
+ #### Шаг 4: R26 находится в зоне 26


#### R23:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R23
!
interface Loopback0
 description R23 Mgmt
 ip address 10.44.44.23 255.255.255.255
 ip router isis 44
!
interface Ethernet0/1
 description p2p to R25
 ip address 10.44.44.1 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R24
 ip address 10.44.44.5 255.255.255.252
 ip router isis 44
!
router isis 44
 net 49.2222.0000.0000.4423.00
!
```

```
R23#show isis neighbors 

Tag 44:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/2       10.44.44.6      UP    8        R24.02             
R25            L1   Et0/1       10.44.44.2      UP    9        R25.01             
R25            L2   Et0/1       10.44.44.2      UP    8        R25.01

R23#show isis database 

Tag 44:
IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00           * 0x00000006   0x577D        535               1/0/0
R25.00-00             0x00000004   0x8B3B        533               1/0/0
R25.01-00             0x00000001   0x1EFD        389               0/0/0
IS-IS Level-2 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00           * 0x00000007   0x6848        529               0/0/0
R24.00-00             0x00000004   0xB43C        614               0/0/0
R24.02-00             0x00000001   0xA005        528               0/0/0
R25.00-00             0x00000005   0x0E9C        615               0/0/0
R25.01-00             0x00000002   0xABF7        1190              0/0/0
R26.00-00             0x00000003   0xD30A        617               0/0/0
R26.01-00             0x00000001   0xCCD4        613               0/0/0
R26.02-00             0x00000001   0xDEC0        614               0/0/0

R23#show isis database verbose 

Tag 44:
IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00           * 0x00000006   0x577D        512               1/0/0
  Area Address: 49.2222
  NLPID:        0xCC 
  Hostname: R23
  IP Address:   10.44.44.23
  Metric: 10         IP 10.44.44.23 255.255.255.255
  Metric: 10         IP 10.44.44.0 255.255.255.252
  Metric: 10         IP 10.44.44.4 255.255.255.252
  Metric: 10         IS R25.01
R25.00-00             0x00000004   0x8B3B        511               1/0/0
  Area Address: 49.2222
  NLPID:        0xCC 
  Hostname: R25
  IP Address:   10.44.44.25
  Metric: 10         IP 10.44.44.25 255.255.255.255
  Metric: 10         IP 10.44.44.0 255.255.255.252
  Metric: 10         IP 10.44.44.12 255.255.255.252
  Metric: 10         IS R25.01
R25.01-00             0x00000002   0x1CFE        1186              0/0/0
  Metric: 0          IS R25.00
  Metric: 0          IS R23.00
IS-IS Level-2 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00           * 0x00000007   0x6848        507               0/0/0
  Area Address: 49.2222
  NLPID:        0xCC 
  Hostname: R23
  IP Address:   10.44.44.23
  Metric: 10         IS R24.02
  Metric: 10         IS R25.01
  Metric: 10         IP 10.44.44.0 255.255.255.252
  Metric: 10         IP 10.44.44.4 255.255.255.252
  Metric: 20         IP 10.44.44.12 255.255.255.252
  Metric: 10         IP 10.44.44.23 255.255.255.255
  Metric: 20         IP 10.44.44.25 255.255.255.255
R24.00-00             0x00000004   0xB43C        591               0/0/0
  Area Address: 49.0024
  NLPID:        0xCC 
  Hostname: R24
  IP Address:   10.44.44.24
  Metric: 10         IS R24.02
  Metric: 10         IS R26.01
  Metric: 10         IP 10.44.44.4 255.255.255.252
  Metric: 10         IP 10.44.44.8 255.255.255.252
  Metric: 10         IP 10.44.44.24 255.255.255.255
R24.02-00             0x00000001   0xA005        506               0/0/0
  Metric: 0          IS R24.00
  Metric: 0          IS R23.00
R25.00-00             0x00000005   0x0E9C        592               0/0/0
  Area Address: 49.2222
  NLPID:        0xCC 
  Hostname: R25
  IP Address:   10.44.44.25
  Metric: 10         IS R25.01
  Metric: 10         IS R26.02
  Metric: 10         IP 10.44.44.0 255.255.255.252
  Metric: 20         IP 10.44.44.4 255.255.255.252
  Metric: 10         IP 10.44.44.12 255.255.255.252
  Metric: 20         IP 10.44.44.23 255.255.255.255
  Metric: 10         IP 10.44.44.25 255.255.255.255
R25.01-00             0x00000002   0xABF7        1167              0/0/0
  Metric: 0          IS R25.00
  Metric: 0          IS R23.00
R26.00-00             0x00000003   0xD30A        594               0/0/0
  Area Address: 49.0026
  NLPID:        0xCC 
  Hostname: R26
  IP Address:   10.44.44.26
  Metric: 10         IS R26.02
  Metric: 10         IS R26.01
  Metric: 10         IP 10.44.44.8 255.255.255.252
  Metric: 10         IP 10.44.44.12 255.255.255.252
  Metric: 10         IP 10.44.44.26 255.255.255.255
R26.01-00             0x00000001   0xCCD4        590               0/0/0
  Metric: 0          IS R26.00
  Metric: 0          IS R24.00
R26.02-00             0x00000001   0xDEC0        591               0/0/0
  Metric: 0          IS R26.00
  Metric: 0          IS R25.00
```

#### R25:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R25
!
interface Loopback0
 description R25 Mgmt
 ip address 10.44.44.25 255.255.255.255
 ip router isis 44
!
interface Ethernet0/1
 description p2p to R23
 ip address 10.44.44.2 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R26
 ip address 10.44.44.13 255.255.255.252
 ip router isis 44
!
router isis 44
 net 49.2222.0000.0000.4425.00
!
```

```
R25#show isis neighbors 

Tag 44:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R23            L1   Et0/1       10.44.44.1      UP    21       R25.01             
R23            L2   Et0/1       10.44.44.1      UP    25       R25.01             
R26            L2   Et0/2       10.44.44.14     UP    9        R26.02   
```

#### R24:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R24
!
interface Loopback0
 description R24 Mgmt
 ip address 10.44.44.24 255.255.255.255
 ip router isis 44
!
interface Ethernet0/1
 description p2p to R26
 ip address 10.44.44.9 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R23
 ip address 10.44.44.6 255.255.255.252
 ip router isis 44
!
router isis 44
 net 49.0024.0000.0000.4424.00
!
```

```
R24#show isis neighbors 

Tag 44:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R23            L2   Et0/2       10.44.44.5      UP    25       R24.02             
R26            L2   Et0/1       10.44.44.10     UP    9        R26.01      
```

#### R26:

```
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R26
!
interface Loopback0
 description R26 Mgmt
 ip address 10.44.44.26 255.255.255.255
 ip router isis 44
!
interface Ethernet0/1
 description p2p to R24
 ip address 10.44.44.10 255.255.255.252
 ip router isis 44
!
interface Ethernet0/2
 description p2p to R25
 ip address 10.44.44.14 255.255.255.252
 ip router isis 44
!
router isis 44
 net 49.0026.0000.0000.4426.00
!
```

```
R26#sh isis neighbors 

Tag 44:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/1       10.44.44.9      UP    26       R26.01             
R25            L2   Et0/2       10.44.44.13     UP    26       R26.02   
```