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
crypto pki trustpoint R19
 enrollment url http://10.111.3.19:80
 ip-address 14.14.14.14
 subject-name CN=R14,OU=R19,O=mydomain,C=RU
 revocation-check none
 rsakeypair R19
!
crypto pki certificate chain R19
 certificate 11
  30820341 30820229 A0030201 02020111 300D0609 2A864886 F70D0101 05050030 
  0D310B30 09060355 04031302 4341301E 170D3233 31323237 31353430 35305A17 
  0D323431 32323631 35343035 305A306A 310B3009 06035504 06130252 55311130 
  0F060355 040A1308 6D79646F 6D61696E 310C300A 06035504 0B130352 3139310C 
  300A0603 55040313 03523134 312C3010 06092A86 4886F70D 01090216 03523134 
  30180609 2A864886 F70D0109 08130B31 342E3134 2E31342E 31343082 0122300D 
  06092A86 4886F70D 01010105 00038201 0F003082 010A0282 010100B0 AE22E791 
  88067B49 FF20D144 F4225587 2C0B4646 4BE4E0A3 250388DD 4507343E 64F830C2 
  353045AA 55A8A06D 273AA922 F7096C98 0A5312F7 A4D4E70B 830F1516 E528A02E 
  B12C83C2 A8F91EF7 A0D088F4 FCBFF3CB 9AF821A9 4E3804AD AF80340C C9B18C08 
  7F90E76C 6FD2FCA3 8840C89E 1E25A2E0 C9EA3893 EA9A8B97 B5B22D22 97C63DAA 
  3B969187 8E6A0391 83EA81B7 7D87FE74 8189BA0E F0ABC648 91ACC1C2 BCA89BE1 
  10DA338E EA401DE6 B5D7233F 2ABFB6C0 E753BC9D B9F3C3DA F7CEB5B3 0940322A 
  54A86733 1EDD7C84 06E2E366 4DDA1304 81B527B0 54FE0DA5 1C6E0B89 6F4B30E0 
  784BF439 57B8B8D7 A06FA200 D20EF5C3 809C86CA 3B16F24E 0A712D02 03010001 
  A34F304D 300B0603 551D0F04 04030205 A0301F06 03551D23 04183016 8014C5FC 
  C997BDDC 52CEFB84 E37E84DC E32A7298 9483301D 0603551D 0E041604 1478D6BC 
  5CDB8A18 49DA7334 A3FC6B42 E76B7072 D3300D06 092A8648 86F70D01 01050500 
  03820101 0007B93D 241FB4A0 7C8D2C24 42DF1F4D 15C0DC55 B421D987 E48C6460 
  74E9B319 297CF670 B6839FBF 7CBBF621 301F456A 4FFB421D 6291F16C 1F3E47B2 
  7A1ACE82 640EDA58 0E4CC76C B882D275 850B0C28 59613EE8 9DE29D8C 3EA0545B 
  3412089D EC2C2FEF AA4EB051 195596E2 89C66EBE D2ABBF06 8690BB67 EC5A86FB 
  56CB0C4C 8ECE5E5F 19AA0273 C214640C ABA2BCE0 23B98F8F FF2AD7DA 180CCEAE 
  1A89C003 68790E83 AE965792 467B6F93 6F0C98AC 809D540F A8674011 021D88F3 
  007A44D7 D1BC3123 7D6B1154 D8E77D16 E832D44F BA11F6FB 8972330A 4A482D2C 
  71EFF61E 4A702B47 08E71C59 69D2F35B B0C6954C 5E05E6D5 9286B9EE F22D8E5F 
  FADCA8AD 56
        quit
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
crypto ikev2 proposal PHASE1 
 encryption aes-cbc-128
 integrity md5
 group 2
!
crypto ikev2 policy IKEV2 
 proposal PHASE1
!
crypto ikev2 profile GRE_PROFILE
 match address local interface Loopback14
 match identity remote address 0.0.0.0 
 authentication remote rsa-sig
 authentication local rsa-sig
 pki trustpoint R19
!
crypto ipsec transform-set GREovIPSEC esp-aes esp-md5-hmac 
 mode tunnel
!
crypto map GRE_over_IPSEC local-address Loopback14
crypto map GRE_over_IPSEC 1 ipsec-isakmp 
 set peer 18.18.18.18
 set transform-set GREovIPSEC 
 set pfs group5
 set ikev2-profile GRE_PROFILE
 match address 118
!
interface Loopback14
 description GRE+DMVPN
 ip address 14.14.14.14 255.255.255.255
!
interface Tunnel14
 description GRE to R18
 ip address 10.111.2.129 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 14.14.14.14
 tunnel destination 18.18.18.18
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
 ip tcp adjust-mss 1360
 tunnel source 14.14.14.14
 tunnel mode gre multipoint
!
interface Ethernet0/2
 description p2p to R15
 ip address 10.111.2.25 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
 crypto map GRE_over_IPSEC
!
interface Ethernet1/0
 description to R22 Kitorn
 ip address 22.111.22.14 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
 crypto map GRE_over_IPSEC
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
ip route 10.112.0.0 255.255.252.0 10.111.2.134
ip route 10.112.0.0 255.255.252.0 10.111.2.130
ip route 40.40.40.40 255.255.255.255 Null0
!
access-list 118 permit gre host 14.14.14.14 host 18.18.18.18
access-list 127 permit gre host 14.14.14.14 host 27.27.27.27
access-list 128 permit gre host 14.14.14.14 host 28.28.28.28
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
crypto pki trustpoint R19
 enrollment url http://10.111.3.19:80
 ip-address 15.15.15.15
 subject-name CN=R15,OU=R19,O=mydomain,C=RU
 revocation-check none
 rsakeypair R19
!
crypto pki certificate chain R19
 certificate 12
  30820341 30820229 A0030201 02020112 300D0609 2A864886 F70D0101 05050030 
  0D310B30 09060355 04031302 4341301E 170D3233 31323237 31353439 32385A17 
  0D323431 32323631 35343932 385A306A 310B3009 06035504 06130252 55311130 
  0F060355 040A1308 6D79646F 6D61696E 310C300A 06035504 0B130352 3139310C 
  300A0603 55040313 03523135 312C3010 06092A86 4886F70D 01090216 03523135 
  30180609 2A864886 F70D0109 08130B31 352E3135 2E31352E 31353082 0122300D 
  06092A86 4886F70D 01010105 00038201 0F003082 010A0282 010100D9 B68A4520 
  8EDA9FFF 6185B783 1B9F7092 BC34DB8E 0936FC7C C71091A1 81B8B6E6 3588895E 
  806B6CCC 4D75ECCF 7CF36106 2AE3BC14 58D4CBD8 A301916F 42A1E76F F6AE3F36 
  4440989F BCE286DC 2D683BF4 1C874487 F27FACB8 E24AE11F 57CAA391 5CAB8C2D 
  9659F08A B87CB869 E543DAE6 A2CDE6E2 C94E30BC 349BC8DA 5D2829D7 DC04B008 
  7C089828 B0FCCFA0 1DDD702D 5F70BB9B 1DF8D539 2CA4BD05 4BAE9F5F BF36158B 
  8CBA8569 04FDA32B 3878A052 855C9126 0FCA5822 0AD9C1EB FAD84C0F C65507D8 
  54D8570A FB1417C8 E4847DA5 B6F4AFE7 BC7D74F5 3B71C7F4 6500A860 D61665CA 
  6F5E517B 1AB72734 C5FF092D D490A217 2E2559E0 29BC5C8D 150DAB02 03010001 
  A34F304D 300B0603 551D0F04 04030205 A0301F06 03551D23 04183016 8014C5FC 
  C997BDDC 52CEFB84 E37E84DC E32A7298 9483301D 0603551D 0E041604 1491E1B8 
  E3380191 AD109711 F90663ED 503499E8 42300D06 092A8648 86F70D01 01050500 
  03820101 0003C053 72D8117E FAFF3267 CC2E5AD7 B7CDD4F2 27293166 F034A8EE 
  213F3BAD 85536E2B 4C5A5427 F9E96762 02F6AF75 567B877E 7853E6DC 80B36B29 
  90642F51 6D761057 431CF510 1606BFC3 02C7FD38 7BBDA695 EC60D1E4 7808C392 
  3A714221 BBDE00CF 16E03286 94F640F1 93872B3A 8F7F35BC EA975A43 FA7CA380 
  DF011E8D C36483B7 920995D5 B5EFBF1C 9A2D7A95 85F67D47 963B4D8F B09E5EB7 
  209B906F 5437FC1B 0EF93F27 83B71F6C 476F1F38 C47830A8 0AB898C2 93309C9D 
  587156CE 378FE1DD 3464B8AD 44C3111A C2795686 D70BD6B2 3C6F3C1D 5963E370 
  97219EEE E6FAB2D2 54279C5C B82C7484 82D66DA6 AF8B3B07 716E8A77 DE0B99AC 
  73EB2D51 C9
        quit
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
crypto ikev2 proposal PHASE1 
 encryption aes-cbc-128
 integrity md5
 group 2
!
crypto ikev2 policy IKEV2 
 proposal PHASE1
!
crypto ikev2 profile GRE_PROFILE
 match address local interface Loopback15
 match identity remote address 0.0.0.0 
 authentication remote rsa-sig
 authentication local rsa-sig
 pki trustpoint R19
!
crypto ipsec transform-set GREovIPSEC esp-aes esp-md5-hmac 
 mode tunnel
!
crypto map GRE_over_IPSEC local-address Loopback15
crypto map GRE_over_IPSEC 1 ipsec-isakmp 
 set peer 18.18.18.18
 set transform-set GREovIPSEC 
 set pfs group5
 set ikev2-profile GRE_PROFILE
 match address 118
!
interface Loopback15
 description GRE+DMVPN
 ip address 15.15.15.15 255.255.255.255
!
interface Tunnel15
 description GRE to R18
 ip address 10.111.2.133 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 15.15.15.15
 tunnel destination 18.18.18.18
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
 ip tcp adjust-mss 1360
 tunnel source 15.15.15.15
 tunnel mode gre multipoint
!
interface Ethernet0/2
 description p2p to R14
 ip address 10.111.2.26 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf network point-to-point
 ip ospf 111 area 0
 crypto map GRE_over_IPSEC
!
interface Ethernet1/0
 description to R21 Lamas
 ip address 33.111.21.15 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
 crypto map GRE_over_IPSEC
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
ip route 10.112.0.0 255.255.252.0 10.111.2.129
ip route 50.50.50.50 255.255.255.255 Null0
!
access-list 118 permit gre host 15.15.15.15 host 18.18.18.18
access-list 127 permit gre host 15.15.15.15 host 27.27.27.27
access-list 128 permit gre host 15.15.15.15 host 28.28.28.28
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
crypto pki trustpoint R19
 enrollment url http://10.111.3.19:80
 ip-address 18.18.18.18
 subject-name CN=R18,OU=R19,O=mydomain,C=RU
 revocation-check none
 rsakeypair R19
!
crypto pki certificate chain R19
 certificate 13
  30820341 30820229 A0030201 02020113 300D0609 2A864886 F70D0101 05050030 
  0D310B30 09060355 04031302 4341301E 170D3233 31323237 31353539 30385A17 
  0D323431 32323631 35353930 385A306A 310B3009 06035504 06130252 55311130 
  0F060355 040A1308 6D79646F 6D61696E 310C300A 06035504 0B130352 3139310C 
  300A0603 55040313 03523138 312C3010 06092A86 4886F70D 01090216 03523138 
  30180609 2A864886 F70D0109 08130B31 382E3138 2E31382E 31383082 0122300D 
  06092A86 4886F70D 01010105 00038201 0F003082 010A0282 010100DD 4CFA2C7E 
  77EBD6DC F1BDE016 F894B96C C43BF0A6 AD53511A D257F19B F725B33C 2E513D2D 
  ACCB9578 B0527515 2A81A56E D39FAAE0 BD2B4023 79F61584 313AF7B1 353497F0 
  3CC7B5E1 705D91F8 73F2A19E E12D284B 4D68545F B41C1F2E 4A4838A7 F9CCD29D 
  F5A960C9 997CA9E9 21E56C7F 6D1250A1 6E4AFB04 6FD99D86 234ABF4A F7783FA0 
  EDDACDFA ACB0EBA4 FB2840AF 747C367B AA6E2A86 C4EAE962 B5E99A4D 2037098A 
  F74A67CA 3B8EA50C 5BB279E3 B2241CF2 F5A6ED5B 077EA9C5 65D086BA EC8EBF5D 
  1D8B7FC4 4FF41224 C2A0490A BD64B46C 053DCB0C 59D300CF F108D9ED EB7FFE4B 
  4125748E DE283634 BE4A776C 0659766A 1519E248 E5FB5F25 EBC24D02 03010001 
  A34F304D 300B0603 551D0F04 04030205 A0301F06 03551D23 04183016 8014C5FC 
  C997BDDC 52CEFB84 E37E84DC E32A7298 9483301D 0603551D 0E041604 147BC2C9 
  3FC07344 655ED1E7 103C4349 84066318 7E300D06 092A8648 86F70D01 01050500 
  03820101 003EED47 93E03AD9 D5C6EC71 F681CF84 14470713 B3088E0E 6D98B5E4 
  C700F68B 12C883D4 E0A9FF71 C7C16C37 32B8122E D6312860 6661F5E7 02822352 
  2E258E88 3E28246F D5533BA4 F2A2F4AE 0BA9D1EA 2CF76809 7071D9E5 A1605CA1 
  1AAB4E28 BE1306AF 1160FFB1 34428FD0 FE123D68 0456707D 1CFF9B5D 759D687D 
  1FFB1CDE 81359D80 7A3FCDC9 2F2AB720 54DB9AFA 5FC3726C 62D42976 37CCABAC 
  58936360 17C9DF1D 927EA501 8995B688 F8B2D2CC F87B06FB 97817C16 461F148D 
  7061F4E7 22BDFC54 C91B67B0 CD45FAD2 530EC4B3 AD882669 850E2434 45F53232 
  3CBD0EC1 CB29061E 688C1477 6836F5F0 4B99C265 AEEEED8C 7A3E7D5C 9FB4319B 
  B085ADB9 8F
        quit
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
crypto ikev2 proposal PHASE1 
 encryption aes-cbc-128
 integrity md5
 group 2
!
crypto ikev2 policy IKEV2 
 proposal PHASE1
!
crypto ikev2 profile GRE_PROFILE
 match address local interface Loopback18
 match identity remote address 0.0.0.0 
 authentication remote rsa-sig
 authentication local rsa-sig
 pki trustpoint R19
!
crypto ipsec transform-set GREovIPSEC esp-aes esp-md5-hmac 
 mode tunnel
!
crypto map GRE_over_IPSEC local-address Loopback18
crypto map GRE_over_IPSEC 1 ipsec-isakmp 
 set peer 14.14.14.14
 set transform-set GREovIPSEC 
 set pfs group5
 set ikev2-profile GRE_PROFILE
 match address 114
crypto map GRE_over_IPSEC 2 ipsec-isakmp 
 set peer 15.15.15.15
 set transform-set GREovIPSEC 
 set pfs group5
 set ikev2-profile GRE_PROFILE
 match address 115
!
interface Loopback18
 description GRE to R14-R15
 ip address 18.18.18.18 255.255.255.255
!
interface Tunnel14
 description GRE to R14
 ip address 10.111.2.130 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 18.18.18.18
 tunnel destination 14.14.14.14
!
interface Tunnel15
 description GRE to R15
 ip address 10.111.2.134 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 3
 tunnel source 18.18.18.18
 tunnel destination 15.15.15.15
!
interface Ethernet1/0
 description to R24 Triada
 ip address 44.112.24.18 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
 crypto map GRE_over_IPSEC
!
interface Ethernet1/1
 description to R26 Triada
 ip address 44.112.26.18 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
 crypto map GRE_over_IPSEC
!
router bgp 2042
 bgp router-id 10.112.3.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 18.18.18.18 mask 255.255.255.255
 network 80.80.80.16 mask 255.255.255.248
 neighbor 44.112.24.24 remote-as 520
 neighbor 44.112.24.24 route-map Triada-OUT out
 neighbor 44.112.26.26 remote-as 520
 neighbor 44.112.26.26 route-map Triada-OUT out
 maximum-paths 2
!
ip route 0.0.0.0 0.0.0.0 44.112.24.24
ip route 0.0.0.0 0.0.0.0 44.112.26.26
ip route 10.111.0.0 255.255.252.0 10.111.2.133
ip route 10.111.0.0 255.255.252.0 10.111.2.129
ip route 10.111.3.19 255.255.255.255 10.111.2.133
ip route 10.111.3.19 255.255.255.255 10.111.2.129
ip route 10.111.4.0 255.255.255.0 10.111.2.133
ip route 10.111.4.0 255.255.255.0 10.111.2.129
ip route 10.111.5.0 255.255.255.0 10.111.2.133
ip route 10.111.5.0 255.255.255.0 10.111.2.129
ip route 80.80.80.16 255.255.255.248 Null0
!
ip prefix-list Triada-OUT seq 5 permit 10.112.0.0/16 le 32
ip prefix-list Triada-OUT seq 6 permit 18.18.18.18/32
ip prefix-list Triada-OUT seq 7 permit 80.80.80.16/29
ip prefix-list Triada-OUT seq 10 deny 0.0.0.0/0 le 32
!
access-list 114 permit gre host 18.18.18.18 host 14.14.14.14
access-list 115 permit gre host 18.18.18.18 host 15.15.15.15
access-list 127 permit gre host 18.18.18.18 host 27.27.27.27
access-list 128 permit gre host 18.18.18.18 host 28.28.28.28
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

### Проверка GRE over IPsec

```
R14# show crypto ikev2 sa    
 IPv4 Crypto IKEv2  SA 

Tunnel-id Local                 Remote                fvrf/ivrf            Status 
1         14.14.14.14/500       18.18.18.18/500       none/none            READY  
      Encr: AES-CBC, keysize: 128, PRF: MD5, Hash: MD596, DH Grp:2, Auth sign: RSA, Auth verify: RSA
      Life/Active Time: 86400/2887 sec

 IPv6 Crypto IKEv2  SA 

R14#
R14#show crypto ipsec sa     

interface: Ethernet0/2
    Crypto map tag: GRE_over_IPSEC, local addr 14.14.14.14

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (14.14.14.14/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (18.18.18.18/255.255.255.255/47/0)
   current_peer 18.18.18.18 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 669, #pkts encrypt: 669, #pkts digest: 669
    #pkts decaps: 769, #pkts decrypt: 769, #pkts verify: 769
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 14.14.14.14, remote crypto endpt.: 18.18.18.18
     plaintext mtu 1438, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/2
     current outbound spi: 0x2D8FE61B(764405275)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x52BEF954(1388247380)
        transform: esp-aes esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 4, flow_id: SW:4, sibling_flags 80000040, crypto map: GRE_over_IPSEC
        sa timing: remaining key lifetime (k/sec): (4305405/687)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x2D8FE61B(764405275)
        transform: esp-aes esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 3, flow_id: SW:3, sibling_flags 80000040, crypto map: GRE_over_IPSEC
        sa timing: remaining key lifetime (k/sec): (4305422/687)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:

interface: Ethernet1/0
    Crypto map tag: GRE_over_IPSEC, local addr 14.14.14.14

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (14.14.14.14/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (18.18.18.18/255.255.255.255/47/0)
   current_peer 18.18.18.18 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 669, #pkts encrypt: 669, #pkts digest: 669
    #pkts decaps: 769, #pkts decrypt: 769, #pkts verify: 769
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 14.14.14.14, remote crypto endpt.: 18.18.18.18
     plaintext mtu 1438, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/2
     current outbound spi: 0x2D8FE61B(764405275)
     PFS (Y/N): N, DH group: none
          
     inbound esp sas:
      spi: 0x52BEF954(1388247380)
        transform: esp-aes esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 4, flow_id: SW:4, sibling_flags 80000040, crypto map: GRE_over_IPSEC
        sa timing: remaining key lifetime (k/sec): (4305405/687)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x2D8FE61B(764405275)
        transform: esp-aes esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 3, flow_id: SW:3, sibling_flags 80000040, crypto map: GRE_over_IPSEC
        sa timing: remaining key lifetime (k/sec): (4305422/687)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:

VPC1> ping 10.112.0.2

84 bytes from 10.112.0.2 icmp_seq=1 ttl=58 time=4.514 ms
84 bytes from 10.112.0.2 icmp_seq=2 ttl=58 time=3.487 ms
84 bytes from 10.112.0.2 icmp_seq=3 ttl=58 time=3.126 ms
84 bytes from 10.112.0.2 icmp_seq=4 ttl=58 time=3.047 ms
84 bytes from 10.112.0.2 icmp_seq=5 ttl=58 time=2.604 ms

VPC1> ping 10.112.1.2

84 bytes from 10.112.1.2 icmp_seq=1 ttl=58 time=3.088 ms
84 bytes from 10.112.1.2 icmp_seq=2 ttl=58 time=2.822 ms
84 bytes from 10.112.1.2 icmp_seq=3 ttl=58 time=3.264 ms
84 bytes from 10.112.1.2 icmp_seq=4 ttl=58 time=3.062 ms
84 bytes from 10.112.1.2 icmp_seq=5 ttl=58 time=3.423 ms

VPC1> trace 10.112.0.2
trace to 10.112.0.2, 8 hops max, press Ctrl+C to stop
 1   10.111.0.1   0.770 ms  0.675 ms  0.440 ms
 2   10.111.2.2   1.009 ms  0.861 ms  0.703 ms
 3   10.111.2.10   0.898 ms  0.788 ms  1.598 ms
 4   10.111.2.130   3.388 ms  2.445 ms  2.303 ms
 5   10.112.2.17   3.012 ms  2.386 ms  2.884 ms
 6   10.112.2.1   2.890 ms  2.682 ms  2.567 ms
 7   *10.112.0.2   4.369 ms (ICMP type:3, code:3, Destination port unreachable)

VPC1> trace 10.112.1.2
trace to 10.112.1.2, 8 hops max, press Ctrl+C to stop
 1   10.111.0.1   1.020 ms  0.698 ms  0.694 ms
 2   10.111.2.2   0.672 ms  1.012 ms  0.729 ms
 3   10.111.2.10   1.050 ms  0.773 ms  0.753 ms
 4   10.111.2.130   3.308 ms  2.308 ms  2.273 ms
 5   10.112.2.21   2.874 ms  2.286 ms  2.397 ms
 6   10.112.2.9   2.503 ms  2.756 ms  2.112 ms
 7   *10.112.1.2   2.985 ms (ICMP type:3, code:3, Destination port unreachable)

VPC7> ping 10.112.0.2

84 bytes from 10.112.0.2 icmp_seq=1 ttl=58 time=2.661 ms
84 bytes from 10.112.0.2 icmp_seq=2 ttl=58 time=2.754 ms
84 bytes from 10.112.0.2 icmp_seq=3 ttl=58 time=2.815 ms
84 bytes from 10.112.0.2 icmp_seq=4 ttl=58 time=2.866 ms
84 bytes from 10.112.0.2 icmp_seq=5 ttl=58 time=2.524 ms

VPC7> ping 10.112.1.2

84 bytes from 10.112.1.2 icmp_seq=1 ttl=58 time=2.830 ms
84 bytes from 10.112.1.2 icmp_seq=2 ttl=58 time=2.632 ms
84 bytes from 10.112.1.2 icmp_seq=3 ttl=58 time=2.842 ms
84 bytes from 10.112.1.2 icmp_seq=4 ttl=58 time=2.747 ms
84 bytes from 10.112.1.2 icmp_seq=5 ttl=58 time=2.698 ms

VPC7> trace 10.112.0.2
trace to 10.112.0.2, 8 hops max, press Ctrl+C to stop
 1   10.111.1.1   0.355 ms  0.314 ms  0.537 ms
 2   10.111.2.6   0.671 ms  0.760 ms  0.675 ms
 3   10.111.2.18   1.432 ms  1.170 ms  0.791 ms
 4   10.111.2.134   2.693 ms  2.315 ms  1.865 ms
 5   10.112.2.17   2.594 ms  2.313 ms  2.277 ms
 6   10.112.2.1   2.517 ms  2.122 ms  2.076 ms
 7   *10.112.0.2   2.354 ms (ICMP type:3, code:3, Destination port unreachable)

VPC7> trace 10.112.1.2
trace to 10.112.1.2, 8 hops max, press Ctrl+C to stop
 1   10.111.1.1   0.547 ms  0.518 ms  0.423 ms
 2   10.111.2.6   1.376 ms  0.669 ms  0.667 ms
 3   10.111.2.18   1.010 ms  1.020 ms  0.702 ms
 4   10.111.2.134   2.541 ms  2.408 ms  3.105 ms
 5   10.112.2.21   2.898 ms  2.248 ms  2.037 ms
 6   10.112.2.9   2.661 ms  2.152 ms  2.182 ms
 7   *10.112.1.2   3.299 ms (ICMP type:3, code:3, Destination port unreachable)

VPC8> ping 10.111.0.2

84 bytes from 10.111.0.2 icmp_seq=1 ttl=58 time=4.477 ms
84 bytes from 10.111.0.2 icmp_seq=2 ttl=58 time=3.746 ms
84 bytes from 10.111.0.2 icmp_seq=3 ttl=58 time=3.552 ms
84 bytes from 10.111.0.2 icmp_seq=4 ttl=58 time=3.260 ms
84 bytes from 10.111.0.2 icmp_seq=5 ttl=58 time=3.990 ms

VPC8> ping 10.111.1.2

84 bytes from 10.111.1.2 icmp_seq=1 ttl=58 time=3.132 ms
84 bytes from 10.111.1.2 icmp_seq=2 ttl=58 time=2.597 ms
84 bytes from 10.111.1.2 icmp_seq=3 ttl=58 time=2.918 ms
84 bytes from 10.111.1.2 icmp_seq=4 ttl=58 time=2.669 ms
84 bytes from 10.111.1.2 icmp_seq=5 ttl=58 time=2.996 ms

VPC8> trace 10.111.0.2
trace to 10.111.0.2, 8 hops max, press Ctrl+C to stop
 1   10.112.0.1   0.562 ms  0.178 ms  0.376 ms
 2   10.112.2.2   0.700 ms  0.506 ms  0.633 ms
 3   10.112.2.18   1.205 ms  1.572 ms  0.530 ms
 4   10.111.2.129   2.138 ms  2.204 ms  1.754 ms
 5   10.111.2.9   2.347 ms  2.125 ms  1.797 ms
 6   10.111.2.1   2.395 ms  1.800 ms  2.035 ms
 7   *10.111.0.2   2.215 ms (ICMP type:3, code:3, Destination port unreachable)

VPC8> trace 10.111.1.2
trace to 10.111.1.2, 8 hops max, press Ctrl+C to stop
 1   10.112.0.1   0.471 ms  0.246 ms  0.355 ms
 2   10.112.2.2   0.579 ms  0.687 ms  0.550 ms
 3   10.112.2.18   1.588 ms  0.784 ms  0.493 ms
 4   10.111.2.129   2.232 ms  1.671 ms  1.688 ms
 5   10.111.2.21   1.891 ms  1.773 ms  2.480 ms
 6   10.111.2.5   2.406 ms  2.160 ms  1.943 ms
 7   *10.111.1.2   2.446 ms (ICMP type:3, code:3, Destination port unreachable)

VPC> ping 10.111.0.2 

84 bytes from 10.111.0.2 icmp_seq=1 ttl=58 time=3.345 ms
84 bytes from 10.111.0.2 icmp_seq=2 ttl=58 time=3.277 ms
84 bytes from 10.111.0.2 icmp_seq=3 ttl=58 time=3.837 ms
84 bytes from 10.111.0.2 icmp_seq=4 ttl=58 time=2.771 ms
84 bytes from 10.111.0.2 icmp_seq=5 ttl=58 time=3.006 ms

VPC> ping 10.111.1.2 

84 bytes from 10.111.1.2 icmp_seq=1 ttl=58 time=4.477 ms
84 bytes from 10.111.1.2 icmp_seq=2 ttl=58 time=2.933 ms
84 bytes from 10.111.1.2 icmp_seq=3 ttl=58 time=3.240 ms
84 bytes from 10.111.1.2 icmp_seq=4 ttl=58 time=3.255 ms
84 bytes from 10.111.1.2 icmp_seq=5 ttl=58 time=2.926 ms

VPC> trace 10.111.0.2
trace to 10.111.0.2, 8 hops max, press Ctrl+C to stop
 1   10.112.1.1   0.224 ms  0.223 ms  0.265 ms
 2   10.112.2.10   0.331 ms  0.417 ms  0.287 ms
 3   10.112.2.22   0.687 ms  1.367 ms  0.791 ms
 4   10.111.2.129   2.676 ms  2.055 ms  2.066 ms
 5   10.111.2.9   1.972 ms  1.939 ms  2.166 ms
 6   10.111.2.1   2.774 ms  2.301 ms  2.213 ms
 7   *10.111.0.2   2.735 ms (ICMP type:3, code:3, Destination port unreachable)

VPC> trace 10.111.1.2
trace to 10.111.1.2, 8 hops max, press Ctrl+C to stop
 1   10.112.1.1   0.262 ms  0.289 ms  0.287 ms
 2   10.112.2.10   0.537 ms  0.357 ms  0.317 ms
 3   10.112.2.22   0.723 ms  0.723 ms  1.018 ms
 4   10.111.2.133   2.520 ms  1.569 ms  1.568 ms
 5   10.111.2.17   1.841 ms  1.944 ms  1.961 ms
 6   10.111.2.5   1.857 ms  2.095 ms  1.893 ms
 7   *10.111.1.2   2.675 ms (ICMP type:3, code:3, Destination port unreachable)
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