# Лабораторная работа 02
+ ## Развертывание коммутируемой сети с резервными каналами
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw01/STP_topology.png?raw=true)

## Цели:
+ ### Часть 1: Создание сети и настройка основных параметров устройств
+ ### Часть 2: Создание VLAN и включение в них портов коммутаторов
+ ### Часть 3: Настройка транков 802.1Q между коммутаторами
+ ### Часть 4: Настройка связности между VLAN на маршрутизаторе
+ ### Часть 5: Проверка функционирования маршрутизации между VLAN

## Решение
## Часть 1: Создание сети и настройка основных параметров устройств

### Шаг 1: Коммутация сети по схеме
### Шаг 2: Настройка основных параметров для каждого коммутатора

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