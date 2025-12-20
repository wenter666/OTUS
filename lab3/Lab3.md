# Д/з №3
## Построение Underlay сети(IS-IS)
Цели занятия
Настроить IS-IS для Underlay сети.

## Сводная таблица (Адресация описана в Lab1)
|  Оборудование  |      Lo0      |
| -------------- |---------------| 
|    Leaf1       | 10.0.255.1/32 | 
|    Leaf2       | 10.0.255.2/32 | 
|    Leaf3       | 10.0.255.3/32 | 
|    Spine1      | 10.0.0.1/32   |
|    Spine2      | 10.0.0.2/32   |

|   Spine ip+1   |      p2p      |   Leaf ip+0    |
|----------------|---------------|----------------|
|     Spine1     |               |                |
|    xe-0/0/1    |  10.0.1.0/31  | xe-0/0/1 leaf1 |
|    xe-0/0/2    |  10.0.1.2/31  | xe-0/0/1 leaf2 |    
|    xe-0/0/3    |  10.0.1.4/31  | xe-0/0/1 leaf3 |
|                |               |                |
|     Spine2     |               |                |
|    xe-0/0/1    |  10.0.2.0/31  | xe-0/0/2 leaf1 |
|    xe-0/0/2    |  10.0.2.2/31  | xe-0/0/2 leaf2 |    
|    xe-0/0/3    |  10.0.2.4/31  | xe-0/0/2 leaf3 |


## Схема
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab2/%D0%A1%D1%85%D0%B5%D0%BC%D1%8B.png)

## Выполнение Juniper
Добавление конфигурации OSPF 
Для всех Leaf одинаковая:
+ set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 interface-type p2p \
set protocols ospf area 0.0.0.0 interface xe-0/0/2.0 interface-type p2p \
set protocols ospf area 0.0.0.0 interface lo0.0 passive

Для всех Spine одинаковая:
+ set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 interface-type p2p \
set protocols ospf area 0.0.0.0 interface xe-0/0/2.0 interface-type p2p \
set protocols ospf area 0.0.0.0 interface xe-0/0/3.0 interface-type p2p \
set protocols ospf area 0.0.0.0 interface lo0.0 passive


Сверка database

root@vQFX-**Leaf1**> show ospf database 

    OSPF database, Area 0.0.0.0
    Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
    Router   10.0.0.1         10.0.0.1         0x80000006  2440  0x22 0xed4c 108
    Router   10.0.0.2         10.0.0.2         0x80000005    77  0x22 0x2909 108
    Router  *10.0.255.1       10.0.255.1       0x80000006  2439  0x22 0xa6e2  84
    Router   10.0.255.2       10.0.255.2       0x80000004    76  0x22 0xf58a  84
    Router   10.0.255.3       10.0.255.3       0x80000004    82  0x22 0x4134  84


root@vQFX-RE-**Leaf2**> show ospf database 

    OSPF database, Area 0.0.0.0
    Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
    Router   10.0.0.1         10.0.0.1         0x80000006   222  0x22 0xed4c 108
    Router   10.0.0.2         10.0.0.2         0x80000004   859  0x22 0x2b08 108
    Router   10.0.255.1       10.0.255.1       0x80000006   223  0x22 0xa6e2  84
    Router  *10.0.255.2       10.0.255.2       0x80000003   856  0x22 0xf789  84
    Router   10.0.255.3       10.0.255.3       0x80000003   864  0x22 0x4333  84


root@vQFX-RE-**Leaf3**> show ospf database 

    OSPF database, Area 0.0.0.0
    Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
    Router   10.0.0.1         10.0.0.1         0x80000006   223  0x22 0xed4c 108
    Router   10.0.0.2         10.0.0.2         0x80000004   860  0x22 0x2b08 108
    Router   10.0.255.1       10.0.255.1       0x80000006   224  0x22 0xa6e2  84
    Router   10.0.255.2       10.0.255.2       0x80000003   859  0x22 0xf789  84
    Router  *10.0.255.3       10.0.255.3       0x80000003   864  0x22 0x4333  84

root@vQFX-RE-**Spine1**> show ospf database 

    OSPF database, Area 0.0.0.0
    Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
    Router  *10.0.0.1         10.0.0.1         0x80000006   224  0x22 0xed4c 108
    Router   10.0.0.2         10.0.0.2         0x80000004   863  0x22 0x2b08 108
    Router   10.0.255.1       10.0.255.1       0x80000006   225  0x22 0xa6e2  84
    Router   10.0.255.2       10.0.255.2       0x80000003   860  0x22 0xf789  84
    Router   10.0.255.3       10.0.255.3       0x80000003   866  0x22 0x4333  84

root@vQFX-RE-**Spine2**> show ospf database 

    OSPF database, Area 0.0.0.0
    Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
    Router   10.0.0.1         10.0.0.1         0x80000006   228  0x22 0xed4c 108
    Router  *10.0.0.2         10.0.0.2         0x80000004   863  0x22 0x2b08 108
    Router   10.0.255.1       10.0.255.1       0x80000006   227  0x22 0xa6e2  84
    Router   10.0.255.2       10.0.255.2       0x80000003   862  0x22 0xf789  84
    Router   10.0.255.3       10.0.255.3       0x80000003   868  0x22 0x4333  84


Пинг Leaf 1 -> Leaf 2/3 \
root@vQFX-Leaf1> ping 10.0.255.2 

    PING 10.0.255.2 (10.0.255.2): 56 data bytes
    64 bytes from 10.0.255.2: icmp_seq=0 ttl=63 time=118.446 ms
    64 bytes from 10.0.255.2: icmp_seq=1 ttl=63 time=120.903 ms
    64 bytes from 10.0.255.2: icmp_seq=2 ttl=63 time=122.598 ms
    64 bytes from 10.0.255.2: icmp_seq=3 ttl=63 time=117.598 ms
    64 bytes from 10.0.255.2: icmp_seq=4 ttl=63 time=288.500 ms

root@vQFX-Leaf1> ping 10.0.255.3  

    PING 10.0.255.3 (10.0.255.3): 56 data bytes
    64 bytes from 10.0.255.3: icmp_seq=0 ttl=63 time=118.517 ms
    64 bytes from 10.0.255.3: icmp_seq=1 ttl=63 time=122.420 ms
    64 bytes from 10.0.255.3: icmp_seq=2 ttl=63 time=118.292 ms
    64 bytes from 10.0.255.3: icmp_seq=3 ttl=63 time=118.164 ms
    64 bytes from 10.0.255.3: icmp_seq=4 ttl=63 time=119.839 ms

*PS: p2p линки тоже пингуются 100ms+, вероятно какая-то проблема виртаулизации*

 Добавленна балансировка 

set policy-options policy-statement LB term LB then load-balance per-packet (На всякий случай. Даная конфа = Per-flow балансировке) \
set routing-options forwarding-table export LB


    root@vQFX-Leaf1> show route forwarding-table destination 10.0.255.2  
    Routing table: default.inet
    Internet:
    Enabled protocols: Bridging, 
    Destination        Type RtRef Next hop           Type Index    NhRef Netif
    10.0.255.2/32      user     0                    ulst   131070     3
                              10.0.1.1           ucst     1734     6 xe-0/0/1.0
                              10.0.2.1           ucst     1735     6 xe-0/0/2.0

    root@vQFX-Leaf1> show route forwarding-table destination 10.0.255.3    
    Routing table: default.inet
    Internet:
    Enabled protocols: Bridging, 
    Destination        Type RtRef Next hop           Type Index    NhRef Netif
    10.0.255.3/32      user     0                    ulst   131070     3
                              10.0.1.1           ucst     1734     6 xe-0/0/1.0
                              10.0.2.1           ucst     1735     6 xe-0/0/2.0

В forwarding-table теперь есть два некст-хопа

## Выполнение Cisco

Добавление конфигурации OSPF 
Для всех Leaf одинаковая (кроме router-id, он равен Lo):
+ feature ospf

  router ospf 0  
    router-id 10.0.255.1

  interface loopback0  
    ip router ospf 0 area 0.0.0.0

  interface Ethernet1/1  
    ip router ospf 0 area 0.0.0.0
    ip ospf network point-to-point

  interface Ethernet1/2  
    ip router ospf 0 area 0.0.0.0
    ip ospf network point-to-point

Для всех Spine одинаковая(кроме router-id, он равен Lo):

+ feature ospf

  router ospf 0  
    router-id 10.0.0.1

  interface loopback0  
    ip router ospf 0 area 0.0.0.0

  interface Ethernet1/1  
    ip router ospf 0 area 0.0.0.0
    ip ospf network point-to-point

  interface Ethernet1/2  
    ip router ospf 0 area 0.0.0.0
    ip ospf network point-to-point

  interface Ethernet1/3  
    ip router ospf 0 area 0.0.0.0
    ip ospf network point-to-point


Сверка database

NXOS-**Leaf1**# sh ip ospf database 

        OSPF Router with ID (10.0.255.1) (Process ID 0 VRF default)

                Router Link States (Area 0.0.0.0)

    Link ID         ADV Router      Age        Seq#       Checksum Link Count
    10.0.0.1        10.0.0.1        171        0x8000000f 0xe183   7   
    10.0.0.2        10.0.0.2        124        0x8000000e 0x1a43   7   
    10.0.255.1      10.0.255.1      128        0x8000000d 0xad57   5   
    10.0.255.2      10.0.255.2      125        0x8000000d 0xc831   5   
    10.0.255.3      10.0.255.3      128        0x8000000b 0x846c   5   


NXOS-**Leaf2**# sh ip ospf  database 

        OSPF Router with ID (10.0.255.2) (Process ID 0 VRF default)

                Router Link States (Area 0.0.0.0)

    Link ID         ADV Router      Age        Seq#       Checksum Link Count
    10.0.0.1        10.0.0.1        227        0x8000000f 0xe183   7   
    10.0.0.2        10.0.0.2        180        0x8000000e 0x1a43   7   
    10.0.255.1      10.0.255.1      186        0x8000000d 0xad57   5   
    10.0.255.2      10.0.255.2      179        0x8000000d 0xc831   5   
    10.0.255.3      10.0.255.3      184        0x8000000b 0x846c   5   


NXOS-**Leaf3**# sh ip ospf database 

        OSPF Router with ID (10.0.255.3) (Process ID 0 VRF default)

                Router Link States (Area 0.0.0.0)

    Link ID         ADV Router      Age        Seq#       Checksum Link Count
    10.0.0.1        10.0.0.1        212        0x8000000f 0xe183   7   
    10.0.0.2        10.0.0.2        165        0x8000000e 0x1a43   7   
    10.0.255.1      10.0.255.1      171        0x8000000d 0xad57   5   
    10.0.255.2      10.0.255.2      167        0x8000000d 0xc831   5   
    10.0.255.3      10.0.255.3      167        0x8000000b 0x846c   5   


NXOS-**Spine1**# sh ip ospf database 

        OSPF Router with ID (10.0.0.1) (Process ID 0 VRF default)

                Router Link States (Area 0.0.0.0)

    Link ID         ADV Router      Age        Seq#       Checksum Link Count
    10.0.0.1        10.0.0.1        290        0x8000000f 0xe183   7   
    10.0.0.2        10.0.0.2        244        0x8000000e 0x1a43   7   
    10.0.255.1      10.0.255.1      248        0x8000000d 0xad57   5   
    10.0.255.2      10.0.255.2      244        0x8000000d 0xc831   5   
    10.0.255.3      10.0.255.3      247        0x8000000b 0x846c   5    

NXOS-**Spine2**# sh ip ospf  database

        OSPF Router with ID (10.0.0.2) (Process ID 0 VRF default)

                Router Link States (Area 0.0.0.0)

    Link ID         ADV Router      Age        Seq#       Checksum Link Count
    10.0.0.1        10.0.0.1        328        0x8000000f 0xe183   7   
    10.0.0.2        10.0.0.2        280        0x8000000e 0x1a43   7   
    10.0.255.1      10.0.255.1      287        0x8000000d 0xad57   5   
    10.0.255.2      10.0.255.2      281        0x8000000d 0xc831   5   
    10.0.255.3      10.0.255.3      284        0x8000000b 0x846c   5   


Пинг Leaf 1 -> Leaf 2/3

    NXOS-Leaf1# ping 10.0.255.2
    PING 10.0.255.2 (10.0.255.2): 56 data bytes
    64 bytes from 10.0.255.2: icmp_seq=0 ttl=253 time=13.909 ms
    64 bytes from 10.0.255.2: icmp_seq=1 ttl=253 time=6.477 ms
    64 bytes from 10.0.255.2: icmp_seq=2 ttl=253 time=5.935 ms
    64 bytes from 10.0.255.2: icmp_seq=3 ttl=253 time=7.834 ms
    64 bytes from 10.0.255.2: icmp_seq=4 ttl=253 time=5.083 ms

    --- 10.0.255.2 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 5.083/7.847/13.909 ms

    NXOS-Leaf1# ping 10.0.255.3
    PING 10.0.255.3 (10.0.255.3): 56 data bytes
    64 bytes from 10.0.255.3: icmp_seq=0 ttl=253 time=18.028 ms
    64 bytes from 10.0.255.3: icmp_seq=1 ttl=253 time=6.097 ms
    64 bytes from 10.0.255.3: icmp_seq=2 ttl=253 time=7.533 ms
    64 bytes from 10.0.255.3: icmp_seq=3 ttl=253 time=6.211 ms
    64 bytes from 10.0.255.3: icmp_seq=4 ttl=253 time=4.907 ms

    --- 10.0.255.3 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 4.907/8.555/18.028 ms



Насколько понимаю ECMP на NEXUS включён по дефолту

    NXOS-Leaf2# sh forwarding 10.0.255.1 detail 
                          Ethernet1/1         


    Prefix 10.0.255.1/32, No of paths: 2, Update time: Wed Dec 17 10:42:35 2025
    Partial Install: No
    10.0.1.3                                  Ethernet1/1         
    10.0.2.3                                  Ethernet1/2       