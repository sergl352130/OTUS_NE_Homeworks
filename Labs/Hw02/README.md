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
### Шаг 3. Включить транковые порты на всех коммутаторах
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
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
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

S1#sh spanning-tree

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
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
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
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
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

S2#sh spanning-tree

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
S3#sh run
Building configuration...
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 spanning-tree cost 90
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
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

S3#sh spanning-tree

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

### С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы:

![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw02/STP_topology_1.png?raw=true)

  1. #### Какой коммутатор является корневым мостом?
     + В данной конфигурации корневым является коммутатор S1
  2. #### Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?
     + Потому что данный коммутатор обладает наименьшим МАС-адресом
  3. #### Какие порты на коммутаторе являются корневыми портами?
     + Корневыми (root) являются порты, которыми коммутатор подключен к корневому мосту
  4. #### Какие порты на коммутаторе являются назначенными портами?
     + Назначенными (designated) являются порты, которые имеют наилучший путь для приема трафика, ведущего к корневому мосту
  5. #### Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?
     + Альтернативными (alternate) становятся порты, которые не являются корневыми или назначенными
  6. #### Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
     + Альтернативные порты блокируются протоколом spanning-tree для предотвращения петель