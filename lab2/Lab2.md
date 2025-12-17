# Д/з №2
## Построение Underlay сети(OSPF)
Цели занятия
Исследовать построение Underlay сети с использованием OSPF.

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