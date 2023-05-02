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