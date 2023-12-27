# Лабораторная работа 14
+ ## IPSec over DMVPN
## Топология
![](https://github.com/sergl352130/OTUS_NE_Homeworks/blob/main/Labs/Hw12/Network_topology.png?raw=true)

## Цели:
+ ### Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
+ ### DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги


### Этапы:
+ #### Шаг 1: Настройка GRE поверх IPSec между офисами Москва и С.-Петербург
+ #### Шаг 2: Настройка DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги
+ #### Шаг 3: Для настройки IPSec использовать CA и сертификаты
+ #### Шаг 4: Все узлы в офисах в лабораторной работе должны иметь IP связность 


### Удостоверяющий центр (CA)

#### R19:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
ip domain name mydomain.ru
!
crypto pki server CA
 no database archive
!
crypto pki trustpoint CA
 revocation-check crl
 rsakeypair CA
!
!
crypto pki certificate chain CA
 certificate ca 01
  308202F8 308201E0 A0030201 02020101 300D0609 2A864886 F70D0101 04050030 
  0D310B30 09060355 04031302 4341301E 170D3233 31323234 31393032 34395A17 
  0D323631 32323331 39303234 395A300D 310B3009 06035504 03130243 41308201 
  22300D06 092A8648 86F70D01 01010500 0382010F 00308201 0A028201 0100CBB1 
  3444E147 4CF668C6 01726913 B5BFB873 83DB15E1 C81756A9 61A36A9E E9E6B393 
  7CF096FF B199CA58 DF67FC83 8BE6DE94 299B49E5 60B4439D 73FC7796 86A98710 
  A3E3B2C8 9D8155B5 81472D36 04AFECC2 1AC38B09 2A2EB32F BA80347B 941F040A 
  8B30D4D2 01A27279 0735D12B 94B3DC42 D715E6F0 754D68A3 FAFD4579 224C2F0B 
  D4D3D943 6ED74016 4BAA40D1 23235813 F2C11214 1589C519 1F309412 D086C780 
  7C14888D 6AD20CEF E002BE48 616787DD 9F2E809D 38C66412 E33B3063 158D6A41 
  8F5639BB 0636825A 9A427352 7D3D991E D92E5D08 E1F261C7 67215FE2 974FBA8D 
  DC156E6E C1FCF917 A28F9CD0 E1F52D8B F82BE9F7 6A2D0599 A94D7E40 35F10203 
  010001A3 63306130 0F060355 1D130101 FF040530 030101FF 300E0603 551D0F01 
  01FF0404 03020186 301F0603 551D2304 18301680 14C5FCC9 97BDDC52 CEFB84E3 
  7E84DCE3 2A729894 83301D06 03551D0E 04160414 C5FCC997 BDDC52CE FB84E37E 
  84DCE32A 72989483 300D0609 2A864886 F70D0101 04050003 82010100 1C3C74C4 
  9A2D59EB 08E087B7 FC64FD5A 7D841BAC 0F2AB5E1 F4DD7728 539D88DD 09BA5959 
  469277A7 FACDF5DD 8B33F6A6 798374E6 A9EDEE93 BF3CC1C5 A9389898 8BFC3687 
  273F034B 7775D704 048BAB4B A41C708E B956F09E 777E2134 1ED484EF E5985303 
  BA4A9508 CD332FBF 763011FA 3C114A7E 7A8B1154 4C5B8E8D 80931FB3 CD8E7B68 
  9F48AC02 41FEC9E8 2C0F7C61 58B0E6FC CE33DD41 6DBBF244 6865227A B5413AB8 
  7556E16B 28D65ABF 823AFE0A 53400F3B 40303F59 C81ACE0F 8F7B3FDD 237C2CD0 
  B395AFBB FB892BBB 5AEF24AE 3A8E89E4 DB65317F 7EEEC680 0E5E3AA5 374C926F 
  8A0A1610 63655F42 98BD5505 F194C0B6 A3E2D11A F8430440 11BF09F2
        quit
!
interface Loopback0
 description R19 Mgmt
 ip address 10.111.3.19 255.255.255.255
 ip ospf 111 area 101
!
```

### Проверка CA

```
R19#show crypto pki server CA
Certificate Server CA:
    Status: enabled
    State: enabled
    Server's configuration is locked  (enter "shut" to unlock it)
    Issuer name: CN=CA
    CA cert fingerprint: 192EA4CF 8565D85C 46023B2D 1AC719F0 
    Granting mode is: manual
    Last certificate issued serial number (hex): 7
    CA certificate expiration timer: 19:02:49 MSK Dec 23 2026
    CRL NextUpdate timer: 15:49:50 MSK Dec 25 2023
    Current primary storage dir: nvram:
    Database Level: Minimum - no cert data written to storage

R19#show crypto pki certificates
CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 19:02:49 MSK Dec 24 2023
    end   date: 19:02:49 MSK Dec 23 2026
  Associated Trustpoints: CA 
  Storage: nvram:CA#1CA.cer

  R14#show crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 11
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R14
    IP Address: 14.14.14.14
    hostname=R14+ipaddress=14.14.14.14
    cn=R14
    ou=R19
    o=mydomain
    c=RU
  Validity Date: 
    start date: 15:40:50 MSK Dec 27 2023
    end   date: 15:40:50 MSK Dec 26 2024
  Associated Trustpoints: R19 
  Storage: nvram:CA#11.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 19:02:49 MSK Dec 24 2023
    end   date: 19:02:49 MSK Dec 23 2026
  Associated Trustpoints: R19 
  Storage: nvram:CA#1CA.cer

  R15#show crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 12
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R15
    IP Address: 15.15.15.15
    hostname=R15+ipaddress=15.15.15.15
    cn=R15
    ou=R19
    o=mydomain
    c=RU
  Validity Date: 
    start date: 15:49:28 MSK Dec 27 2023
    end   date: 15:49:28 MSK Dec 26 2024
  Associated Trustpoints: R19 
  Storage: nvram:CA#12.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 19:02:49 MSK Dec 24 2023
    end   date: 19:02:49 MSK Dec 23 2026
  Associated Trustpoints: R19 
  Storage: nvram:CA#1CA.cer

  R18#show crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 13
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R18
    IP Address: 18.18.18.18
    hostname=R18+ipaddress=18.18.18.18
    cn=R18
    ou=R19
    o=mydomain
    c=RU
  Validity Date: 
    start date: 15:59:08 MSK Dec 27 2023
    end   date: 15:59:08 MSK Dec 26 2024
  Associated Trustpoints: R19 
  Storage: nvram:CA#13.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 19:02:49 MSK Dec 24 2023
    end   date: 19:02:49 MSK Dec 23 2026
  Associated Trustpoints: R19 
  Storage: nvram:CA#1CA.cer

R27#show crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 14
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R27
    IP Address: 27.27.27.27
    hostname=R27+ipaddress=27.27.27.27
    cn=R27
    ou=R19
    o=mydomain
    c=RU
  Validity Date: 
    start date: 19:11:25 MSK Dec 27 2023
    end   date: 19:11:25 MSK Dec 26 2024
  Associated Trustpoints: R19 
  Storage: nvram:CA#14.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 22:02:49 MSK Dec 24 2023
    end   date: 22:02:49 MSK Dec 23 2026
  Associated Trustpoints: R19 
  Storage: nvram:CA#1CA.cer

R28#show crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 15
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R28
    IP Address: 28.28.28.28
    hostname=R28+ipaddress=28.28.28.28
    cn=R28
    ou=R19
    o=mydomain
    c=RU
  Validity Date: 
    start date: 16:21:45 MSK Dec 27 2023
    end   date: 16:21:45 MSK Dec 26 2024
  Associated Trustpoints: R19 
  Storage: nvram:CA#15.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 19:02:49 MSK Dec 24 2023
    end   date: 19:02:49 MSK Dec 23 2026
  Associated Trustpoints: R19 
  Storage: nvram:CA#1CA.cer
```


### Настройка GRE и DMVPN over IPsec


#### R14:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
interface Loopback18
 description GRE+DMVPN
 ip address 40.40.40.40 255.255.255.255
!
interface Tunnel18
 description GRE to R18
 ip address 10.111.2.129 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 40.40.40.40
 tunnel destination 80.80.80.80
!
interface Tunnel114
 description DMVPN to R27-R28
 ip address 10.111.2.194 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp registration timeout 60
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source 40.40.40.40
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.111.3.14
 bgp log-neighbor-changes
 network 10.111.3.19 mask 255.255.255.255
 network 14.14.14.14 mask 255.255.255.255
 network 40.40.40.40 mask 255.255.255.255
 neighbor 10.111.3.15 remote-as 1001
 neighbor 10.111.3.15 update-source Loopback0
 neighbor 10.111.3.15 next-hop-self
 neighbor 22.111.22.22 remote-as 101
 neighbor 22.111.22.22 route-map R22-Kitorn-OUT out
!
ip route 10.111.4.0 255.255.255.0 10.111.2.208
ip route 10.111.5.0 255.255.255.0 10.111.2.207
ip route 10.112.0.0 255.255.252.0 10.111.2.130
ip route 14.14.14.14 255.255.255.255 Null0
!
```

#### R15:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R15
!
interface Loopback18
 description GRE+DMVPN
 ip address 50.50.50.50 255.255.255.255
!
interface Tunnel18
 description GRE to R18
 ip address 10.111.2.133 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 50.50.50.50
 tunnel destination 80.80.80.80
!
interface Tunnel115
 description DMVPN to R27-R28
 ip address 10.111.2.225 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp registration timeout 60
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source 50.50.50.50
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.111.3.15
 bgp log-neighbor-changes
 network 15.15.15.15 mask 255.255.255.255
 network 50.50.50.50 mask 255.255.255.255
 neighbor 10.111.3.14 remote-as 1001
 neighbor 10.111.3.14 update-source Loopback0
 neighbor 10.111.3.14 next-hop-self
 neighbor 33.111.21.21 remote-as 301
 neighbor 33.111.21.21 route-map R21-Lamas-IN in
 neighbor 33.111.21.21 route-map R21-Lamas-OUT out
!
ip route 10.111.4.0 255.255.255.0 10.111.2.228
ip route 10.111.5.0 255.255.255.0 10.111.2.227
ip route 10.112.0.0 255.255.252.0 10.111.2.134
ip route 15.15.15.15 255.255.255.255 Null0
!
```

#### R18:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R18
!
interface Loopback18
 description GRE to R14-R15
 ip address 80.80.80.80 255.255.255.255
!
interface Tunnel14
 description GRE to R14
 ip address 10.111.2.130 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 80.80.80.80
 tunnel destination 40.40.40.40
!
interface Tunnel15
 description GRE to R15
 ip address 10.111.2.134 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 80.80.80.80
 tunnel destination 50.50.50.50
!
router bgp 2042
 bgp router-id 10.112.3.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 10.112.3.18 mask 255.255.255.255
 network 18.18.18.16 mask 255.255.255.248
 network 80.80.80.80 mask 255.255.255.255
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26
ip route 10.111.0.0 255.255.252.0 10.111.2.129
ip route 10.111.0.0 255.255.252.0 10.111.2.133
ip route 18.18.18.16 255.255.255.248 Null0
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 6 permit 18.18.18.16/29
ip prefix-list Triada-OUT seq 7 permit 80.80.80.80/32
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
```

#### R27:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R27
!
interface Loopback27
 description DMVPN
 ip address 27.27.27.27 255.255.255.255
!
interface Tunnel114
 description DMVPN to R14
 ip address 10.111.2.207 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 40.40.40.40
 ip nhrp map 10.111.2.194 40.40.40.40
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.194
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 27.27.27.27
 tunnel mode gre multipoint
!
interface Tunnel115
 description DMVPN to R15
 ip address 10.111.2.227 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 50.50.50.50
 ip nhrp map 10.111.2.225 50.50.50.50
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.225
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 27.27.27.27
 tunnel mode gre multipoint
!
ip route 0.0.0.0 0.0.0.0 44.115.25.25
ip route 10.111.0.0 255.255.252.0 10.111.2.194
ip route 10.111.0.0 255.255.252.0 10.111.2.225
ip route 10.111.4.0 255.255.255.0 10.111.2.194
ip route 10.111.4.0 255.255.255.0 10.111.2.225
!
```


#### R28:

```
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R28
!
interface Loopback28
 description DMVPN
 ip address 28.28.28.28 255.255.255.255
!
interface Tunnel114
 description DMVPN to R14
 ip address 10.111.2.208 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 40.40.40.40
 ip nhrp map 10.111.2.194 40.40.40.40
 ip nhrp network-id 114
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.194
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 28.28.28.28
 tunnel mode gre multipoint
!
interface Tunnel115
 description DMVPN to R15
 ip address 10.111.2.228 255.255.255.224
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 50.50.50.50
 ip nhrp map 10.111.2.225 50.50.50.50
 ip nhrp network-id 115
 ip nhrp holdtime 180
 ip nhrp nhs 10.111.2.225
 ip nhrp registration timeout 60
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source 28.28.28.28
 tunnel mode gre multipoint
!
ip route 0.0.0.0 0.0.0.0 44.114.25.25
ip route 0.0.0.0 0.0.0.0 44.114.26.26
ip route 10.111.0.0 255.255.252.0 10.111.2.194
ip route 10.111.0.0 255.255.252.0 10.111.2.225
ip route 10.111.5.0 255.255.255.0 10.111.2.194
ip route 10.111.5.0 255.255.255.0 10.111.2.225
!
```

### Проверка GRE

```
R14#show interfaces tunnel 18
Tunnel18 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R18
  Internet address is 10.111.2.129/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 40.40.40.40, destination 80.80.80.80
  Tunnel protocol/transport GRE/IP

R15#show interfaces tunnel 18
Tunnel18 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R18
  Internet address is 10.111.2.133/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 50.50.50.50, destination 80.80.80.80
  Tunnel protocol/transport GRE/IP

R18#show interfaces tunnel 14
Tunnel14 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R14
  Internet address is 10.111.2.130/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 80.80.80.80, destination 40.40.40.40
  Tunnel protocol/transport GRE/IP

R18#show interfaces tunnel 15
Tunnel15 is up, line protocol is up 
  Hardware is Tunnel
  Description: GRE to R15
  Internet address is 10.111.2.134/30
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive set (10 sec), retries 3
  Tunnel linestate evaluation up
  Tunnel source 80.80.80.80, destination 50.50.50.50
  Tunnel protocol/transport GRE/IP

VPC1> ping 10.112.0.2

84 bytes from 10.112.0.2 icmp_seq=1 ttl=58 time=2.767 ms
84 bytes from 10.112.0.2 icmp_seq=2 ttl=58 time=2.749 ms
84 bytes from 10.112.0.2 icmp_seq=3 ttl=58 time=2.575 ms
84 bytes from 10.112.0.2 icmp_seq=4 ttl=58 time=2.920 ms
84 bytes from 10.112.0.2 icmp_seq=5 ttl=58 time=2.930 ms

VPC1> ping 10.112.1.2

84 bytes from 10.112.1.2 icmp_seq=1 ttl=58 time=2.561 ms
84 bytes from 10.112.1.2 icmp_seq=2 ttl=58 time=2.829 ms
84 bytes from 10.112.1.2 icmp_seq=3 ttl=58 time=2.302 ms
84 bytes from 10.112.1.2 icmp_seq=4 ttl=58 time=4.464 ms
84 bytes from 10.112.1.2 icmp_seq=5 ttl=58 time=2.231 ms

VPC7> ping 10.112.0.2

84 bytes from 10.112.0.2 icmp_seq=1 ttl=58 time=4.005 ms
84 bytes from 10.112.0.2 icmp_seq=2 ttl=58 time=2.347 ms
84 bytes from 10.112.0.2 icmp_seq=3 ttl=58 time=3.562 ms
84 bytes from 10.112.0.2 icmp_seq=4 ttl=58 time=2.715 ms
84 bytes from 10.112.0.2 icmp_seq=5 ttl=58 time=2.582 ms

VPC7> ping 10.112.1.2

84 bytes from 10.112.1.2 icmp_seq=1 ttl=58 time=4.192 ms
84 bytes from 10.112.1.2 icmp_seq=2 ttl=58 time=3.015 ms
84 bytes from 10.112.1.2 icmp_seq=3 ttl=58 time=2.758 ms
84 bytes from 10.112.1.2 icmp_seq=4 ttl=58 time=2.994 ms
84 bytes from 10.112.1.2 icmp_seq=5 ttl=58 time=2.551 ms

```

### Проверка DMVPN

```
R14#show interfaces tunnel 114
Tunnel114 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R27-R28
  Internet address is 10.111.2.194/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 40.40.40.40
  Tunnel protocol/transport multi-GRE/IP

R14#
R14#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel114, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 27.27.27.27        10.111.2.207    UP 00:54:58     D
     1 28.28.28.28        10.111.2.208    UP 00:54:27     D

R14#
R14#sh ip nhrp
10.111.2.207/32 via 10.111.2.207
   Tunnel114 created 00:55:09, expire 00:02:24
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 27.27.27.27 
10.111.2.208/32 via 10.111.2.208
   Tunnel114 created 00:54:37, expire 00:02:30
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 28.28.28.28 


R15#show interfaces tunnel 115
Tunnel115 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R27-R28
  Internet address is 10.111.2.225/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 50.50.50.50
  Tunnel protocol/transport multi-GRE/IP

R15#
R15#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel115, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 27.27.27.27        10.111.2.227    UP 00:55:27     D
     1 28.28.28.28        10.111.2.228    UP 00:55:48     D

R15#
R15#sh ip nhrp
10.111.2.227/32 via 10.111.2.227
   Tunnel115 created 00:55:35, expire 00:02:49
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 27.27.27.27 
10.111.2.228/32 via 10.111.2.228
   Tunnel115 created 00:55:56, expire 00:02:15
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 28.28.28.28


R27#show interfaces tunnel 114
Tunnel114 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R14
  Internet address is 10.111.2.207/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 27.27.27.27
  Tunnel protocol/transport multi-GRE/IP

R27#
R27#show interfaces tunnel 115
Tunnel115 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R15
  Internet address is 10.111.2.227/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 27.27.27.27
  Tunnel protocol/transport multi-GRE/IP

R27#
R27#show dmvpn       
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel114, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 40.40.40.40        10.111.2.194  NHRP 03:32:03     S
     0 UNKNOWN            10.111.2.208  NHRP    never    IX

Interface: Tunnel115, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 50.50.50.50        10.111.2.225    UP 03:28:17     S
     1 28.28.28.28        10.111.2.228    UP 00:00:30     D

R27#
R27#show ip nhrp
10.111.2.194/32 via 10.111.2.194
   Tunnel114 created 01:08:38, never expire 
   Type: static, Flags: used 
   NBMA address: 40.40.40.40 
10.111.2.225/32 via 10.111.2.225
   Tunnel115 created 01:08:38, never expire 
   Type: static, Flags: used 
   NBMA address: 50.50.50.50 


R28#show interfaces tunnel 114
Tunnel114 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R14
  Internet address is 10.111.2.208/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 28.28.28.28
  Tunnel protocol/transport multi-GRE/IP

R28#
R28#show interfaces tunnel 115
Tunnel115 is up, line protocol is up 
  Hardware is Tunnel
  Description: DMVPN to R15
  Internet address is 10.111.2.228/27
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 28.28.28.28
  Tunnel protocol/transport multi-GRE/IP

R28#
R28#show dmvpn       
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel114, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 40.40.40.40        10.111.2.194  NHRP 02:24:26     S
     0 UNKNOWN            10.111.2.207  NHRP    never    IX

Interface: Tunnel115, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 50.50.50.50        10.111.2.225    UP 02:25:15     S
     1 27.27.27.27        10.111.2.227    UP 00:00:37     D


R28#
R28#show ip nhrp
10.111.2.194/32 via 10.111.2.194
   Tunnel114 created 02:10:44, never expire 
   Type: static, Flags: 
   NBMA address: 40.40.40.40 
10.111.2.225/32 via 10.111.2.225
   Tunnel115 created 02:09:18, never expire 
   Type: static, Flags: 
   NBMA address: 50.50.50.50


VPC1> ping 10.111.4.65 

84 bytes from 10.111.4.65 icmp_seq=1 ttl=252 time=2.452 ms
84 bytes from 10.111.4.65 icmp_seq=2 ttl=252 time=2.263 ms
84 bytes from 10.111.4.65 icmp_seq=3 ttl=252 time=2.658 ms
84 bytes from 10.111.4.65 icmp_seq=4 ttl=252 time=2.251 ms
84 bytes from 10.111.4.65 icmp_seq=5 ttl=252 time=1.989 ms

VPC1> ping 10.111.4.129

84 bytes from 10.111.4.129 icmp_seq=1 ttl=252 time=2.574 ms
84 bytes from 10.111.4.129 icmp_seq=2 ttl=252 time=2.709 ms
84 bytes from 10.111.4.129 icmp_seq=3 ttl=252 time=2.896 ms
84 bytes from 10.111.4.129 icmp_seq=4 ttl=252 time=2.622 ms
84 bytes from 10.111.4.129 icmp_seq=5 ttl=252 time=2.671 ms

VPC1> ping 10.111.5.129

84 bytes from 10.111.5.129 icmp_seq=1 ttl=252 time=3.186 ms
84 bytes from 10.111.5.129 icmp_seq=2 ttl=252 time=3.304 ms
84 bytes from 10.111.5.129 icmp_seq=3 ttl=252 time=2.546 ms
84 bytes from 10.111.5.129 icmp_seq=4 ttl=252 time=2.521 ms
84 bytes from 10.111.5.129 icmp_seq=5 ttl=252 time=8.531 ms


VPC7> ping 10.111.4.65

84 bytes from 10.111.4.65 icmp_seq=1 ttl=252 time=2.718 ms
84 bytes from 10.111.4.65 icmp_seq=2 ttl=252 time=2.804 ms
84 bytes from 10.111.4.65 icmp_seq=3 ttl=252 time=2.426 ms
84 bytes from 10.111.4.65 icmp_seq=4 ttl=252 time=2.278 ms
84 bytes from 10.111.4.65 icmp_seq=5 ttl=252 time=2.668 ms

VPC7> ping 10.111.4.129

84 bytes from 10.111.4.129 icmp_seq=1 ttl=252 time=1.996 ms
84 bytes from 10.111.4.129 icmp_seq=2 ttl=252 time=2.292 ms
84 bytes from 10.111.4.129 icmp_seq=3 ttl=252 time=2.090 ms
84 bytes from 10.111.4.129 icmp_seq=4 ttl=252 time=3.190 ms
84 bytes from 10.111.4.129 icmp_seq=5 ttl=252 time=3.112 ms

VPC7> ping 10.111.5.129

84 bytes from 10.111.5.129 icmp_seq=1 ttl=252 time=2.980 ms
84 bytes from 10.111.5.129 icmp_seq=2 ttl=252 time=2.995 ms
84 bytes from 10.111.5.129 icmp_seq=3 ttl=252 time=3.127 ms
84 bytes from 10.111.5.129 icmp_seq=4 ttl=252 time=2.405 ms
84 bytes from 10.111.5.129 icmp_seq=5 ttl=252 time=2.908 ms

```