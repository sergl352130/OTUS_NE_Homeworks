# Лабораторная работа 02
+ ## Развертывание коммутируемой сети с резервными каналами
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw02/STP_topology.png?raw=true)

## Цели:
+ ### Часть 1. Создание сети и настройка основных параметров устройства
+ ### Часть 2. Выбор корневого моста
+ ### Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
+ ### Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

## Решение
## Часть 1: Создание сети и настройка основных параметров устройств

### Шаг 1: Коммутация сети по схеме
### Шаг 2: Инициализация и перезагрузка коммутаторов
### Шаг 3: Настройка основных параметров для каждого коммутатора

### S1:

```
S1#sh run
Building configuration...
!
hostname S1
!
!
enable secret 5 $1$iyZ6$r63d1wdyvn6Jl8V8CZTU8.
!
!
no ip domain-lookup
!
!
interface Vlan1
 ip address 192.168.1.1 255.255.255.0
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 030752180500
 logging synchronous
line aux 0
line vty 0 4
 password 7 0822455D0A16
 login
!
!
end
```

### S2:

```
S2#sh run
Building configuration...
!
hostname S2
!
!
enable secret 5 $1$pHN4$2MyeOcN1q4lZN0rr84jQu0
!
!
no ip domain-lookup
!
!
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 104D000A0618
 logging synchronous
line aux 0
line vty 0 4
 password 7 00071A150754
 login
!
!
end
```

### S3:

```
S3#sh run
Building configuration...
!
hostname S3
!
!
enable secret 5 $1$2TMM$RimE0i6g1XLZrULzCrtcX/
!
!
no ip domain-lookup
!
!
interface Vlan1
 ip address 192.168.1.3 255.255.255.0
!
!
banner motd ^CThis is a secure system. Unauthorized access is prohibited!^C
!
line con 0
 password 7 030752180500
 logging synchronous
line aux 0
line vty 0 4
 password 7 045802150C2E
 login
!
!
end
```

### Шаг 4: Проверка связности

### S1:

```
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### S2:

```
S2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

## Часть 2. Определение корневого моста

### Шаг 1. Отключить все порты на коммутаторах
### Шаг 2. Настроить подключенные порты в качестве транковых
### Шаг 3. Включить порты E0/1 и E0/3 на всех коммутаторах
### Шаг 4. Отобразить данные протокола spanning-tree

### S1:

```
S1#sh run
Building configuration...
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Vlan1
 ip address 192.168.1.1 255.255.255.0
!
!
end

S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr
```

### S2:

```
S2#sh run
Building configuration...
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
!
!
end

S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr
```

### S3:

```
S3#sh run
Building configuration...
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Vlan1
 ip address 192.168.1.3 255.255.255.0
!
!
end

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr
```

### С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw02/STP_topology_1.png?raw=true)

  1. #### Какой коммутатор является корневым мостом?
     + В данной конфигурации корневым является коммутатор S1.
  2. #### Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?
     + Потому что у него наименьший Bridge ID (BID = Bridge Priority + System Extension + MAC). Так как в этой сети Bridge Priority у всех коммутаторов одинаковый (32768), sys-id-ext, который равен VLAN ID, тоже одинаковый (1), наименьший BID у коммутатора S1 с наименьшим МАС-адресом (aabb.cc00.1000).
  3. #### Какие порты на коммутаторе являются корневыми портами?
     + Корневыми (root) являются порты, ближайшие к корневому мосту по общей стоимости (cost) пути к нему.
  4. #### Какие порты на коммутаторе являются назначенными портами?
     + Назначенными (designated) являются порты, которые имеют наилучший путь для приема трафика, ведущего к корневому мосту.
  5. #### Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?
     + Альтернативными (alternate) становятся порты, которые не являются корневыми или назначенными.
  6. #### Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
     + Учитывая, что стоимости (cost) портов S2:E0/3 и S3:E0/3 одинаковы, при выборе designated порта был принят во внимание Bridge ID, который, как сказано в п.2, включает в себя МАС-адрес. У коммутатора S2 меньше МАС-адрес (aabb.cc00.2000), следоветаельно меньше BID. Соответственно порт Е0/3 коммутатора S2 был выбран designated, а порт Е0/3 коммутатора S3 стал alternate и был заблокирован.


## Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов

### Шаг 1. Определить коммутатор с заблокированным портом
### Шаг 2. Изменить стоимость root порта

### S3:

```
S3#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
S3(config)#int Et0/1
S3(config-if)#spanning-tree cost 90
S3(config-if)#^Z
S3#
S3#sh run
Building configuration...
!
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 spanning-tree cost 90
!
!
end
```

### Шаг 3. Просмотреть изменения протокола spanning-tree

### S3:

```
S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        90
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 90        128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr
```

### S2:

```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr
```

### Ответ на вопрос:

  1. #### Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?
     + Это происходит потому, что стоимости (cost) портов являются более приоритетными. Соответственно когда стоимость пути к root bridge (S1) от порта S3:E0/3 административно меняется со значения 200 (100+100) на 190 (100+90), порт Е0/3 коммутатора S3 в результате выбирается designated, так как стоимость пути к S1 от порта S2:E0/3 остается равным 200 (100+100). При этом порт Е0/3 коммутатора S2 становится alternate и блокируется.

### Шаг 4. Удалить изменения стоимости порта

### S3:

```
S3#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
S3(config)#int Et0/1
S3(config-if)#no spanning-tree cost 90
S3(config-if)#^Z
S3#
S3#sh run
Building configuration...
!
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
!
end

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr
```

### S2:

```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr
```

## Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

### Шаг 1. Включить порты E0/0 и E0/2 на всех коммутаторах
### Шаг 2. Подождать 30 секунд, чтобы протокол STP завершил процесс перевода порта, после чего выполнить команду show spanning-tree на коммутаторах некорневого моста. Обратите внимание, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста

### S2:

```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
```

### S3:

```
S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Altn BLK 100       128.3    Shr
Et0/3               Altn BLK 100       128.4    Shr
```

### Ответы на вопросы:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw02/STP_topology_2.png?raw=true)

  1. #### Какой порт выбран протоколом STP в качестве корневого на каждом коммутаторе некорневого моста?
     + S3: E0/0 вместо E0/1; S2: E0/0 вместо E0/1.
  2. #### Почему протокол STP выбрал эти порты в качестве корневых?
     + Потому что ротоколом STP принимается во внимание параметр Port ID, который включает Port Priority и номер порта. Port Priority одинаковые, а в целом Port ID меньший у портов S2:E0/0 и S2:E0/0, так как они подключены к портам корневого моста с наименьшими номерами (по схеме Et0/0 для связи с S2, и Et0/2 для связи с S3)
  


## Вопросы для повторения:

  1. #### Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
     + Bridge ID
  2. #### Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
     + Port Priority
  3. #### Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
     + Port ID  