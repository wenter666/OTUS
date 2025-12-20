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
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab3/%D0%A1%D1%85%D0%B5%D0%BC%D1%8B.png)

## Сводная таблица NET
|  Оборудование  |      Lo0      |            NET            |
| -------------- |---------------|---------------------------|
|    Leaf1       | 10.0.255.1/32 | 49.0001.0010.0025.5001.00 |
|    Leaf2       | 10.0.255.2/32 | 49.0001.0010.0025.5002.00 |
|    Leaf3       | 10.0.255.3/32 | 49.0001.0010.0025.5003.00 |
|    Spine1      | 10.0.0.1/32   | 49.0001.0010.0000.0001.00 |
|    Spine2      | 10.0.0.2/32   | 49.0001.0010.0000.0002.00 |


## Выполнение Juniper
Добавление конфигурации IS-IS  
Для всех Leaf одинаковая (Кроме NET идентификатора):

+ set protocols isis interface lo0.0 passive  
set protocols isis level 1 disable  
set protocols isis reference-bandwidth 10g  
set protocols isis level 2 wide-metrics-only  
set protocols isis interface xe-0/0/2.0 point-to-point  
set protocols isis interface xe-0/0/1.0 point-to-point  
set protocols isis interface lo0.0 passive

+ set interfaces xe-0/0/1 unit 0 family iso  
set interfaces xe-0/0/2 unit 0 family iso  
set interfaces lo0 unit 0 family iso address 49.0001.0010.0025.5001.00 

Для всех Spine одинаковая (Кроме NET идентификатора):
+ set protocols isis interface lo0.0 passive  
set protocols isis level 1 disable  
set protocols isis reference-bandwidth 10g  
set protocols isis level 2 wide-metrics-only  
set protocols isis interface xe-0/0/3.0 point-to-point  
set protocols isis interface xe-0/0/2.0 point-to-point  
set protocols isis interface xe-0/0/1.0 point-to-point  
set protocols isis interface lo0.0 passive

+ set interfaces xe-0/0/1 unit 0 family iso  
set interfaces xe-0/0/2 unit 0 family iso  
set interfaces xe-0/0/3 unit 0 family iso  
set interfaces lo0 unit 0 family iso address 49.0001.0010.0000.0001.00 


Сверка database

root@vQFX-RE-**Leaf1**> show isis database 

    IS-IS level 1 link-state database:
      0 LSPs

    IS-IS level 2 link-state database:
    LSP ID                      Sequence Checksum Lifetime Attributes
    vQFX-RE-Spine1.00-00             0x1   0x6325     1174 L1 L2
    vQFX-RE-Spine2.00-00             0x1   0x1861     1174 L1 L2
    vQFX-RE-Leaf1.00-00              0x1   0x27cc     1176 L1 L2
    vQFX-RE-Leaf2.00-00              0x1   0x21c0     1173 L1 L2
    vQFX-RE-Leaf3.00-00              0x1   0x1bb4     1172 L1 L2
      5 LSPs


root@vQFX-RE-**Leaf2**> show isis database

    IS-IS level 1 link-state database:
      0 LSPs

    IS-IS level 2 link-state database:
    LSP ID                      Sequence Checksum Lifetime Attributes
    vQFX-RE-Spine1.00-00             0x1   0x6325     1174 L1 L2
    vQFX-RE-Spine2.00-00             0x1   0x1861     1174 L1 L2
    vQFX-RE-Leaf1.00-00              0x1   0x27cc     1172 L1 L2
    vQFX-RE-Leaf2.00-00              0x1   0x21c0     1176 L1 L2
    vQFX-RE-Leaf3.00-00              0x1   0x1bb4     1172 L1 L2
      5 LSPs


root@vQFX-RE-**Leaf3**> show isis database

    IS-IS level 1 link-state database:
      0 LSPs

    IS-IS level 2 link-state database:
    LSP ID                      Sequence Checksum Lifetime Attributes
    vQFX-RE-Spine1.00-00             0x1   0x6325     1174 L1 L2
    vQFX-RE-Spine2.00-00             0x1   0x1861     1174 L1 L2
    vQFX-RE-Leaf1.00-00              0x1   0x27cc     1172 L1 L2
    vQFX-RE-Leaf2.00-00              0x1   0x21c0     1173 L1 L2
    vQFX-RE-Leaf3.00-00              0x1   0x1bb4     1176 L1 L2
      5 LSPs


root@vQFX-RE-**Spine1**> show isis database

    IS-IS level 1 link-state database:
      0 LSPs

    IS-IS level 2 link-state database:
    LSP ID                      Sequence Checksum Lifetime Attributes
    vQFX-RE-Spine1.00-00             0x1   0x6325     1176 L1 L2
    vQFX-RE-Spine2.00-00             0x1   0x1861     1172 L1 L2
    vQFX-RE-Leaf1.00-00              0x1   0x27cc     1174 L1 L2
    vQFX-RE-Leaf2.00-00              0x1   0x21c0     1174 L1 L2
    vQFX-RE-Leaf3.00-00              0x1   0x1bb4     1174 L1 L2
      5 LSPs

root@vQFX-RE-**Spine2**> show isis database

    IS-IS level 1 link-state database:
      0 LSPs

    IS-IS level 2 link-state database:
    LSP ID                      Sequence Checksum Lifetime Attributes
    vQFX-RE-Spine1.00-00             0x1   0x6325     1172 L1 L2
    vQFX-RE-Spine2.00-00             0x1   0x1861     1176 L1 L2
    vQFX-RE-Leaf1.00-00              0x1   0x27cc     1174 L1 L2
    vQFX-RE-Leaf2.00-00              0x1   0x21c0     1174 L1 L2
    vQFX-RE-Leaf3.00-00              0x1   0x1bb4     1174 L1 L2
      5 LSPs



Пинг Leaf 1 -> Leaf 2/3 

    root@vQFX-RE-Leaf1> ping 10.0.255.2
    PING 10.0.255.2 (10.0.255.2): 56 data bytes
    64 bytes from 10.0.255.2: icmp_seq=0 ttl=63 time=217.033 ms
    64 bytes from 10.0.255.2: icmp_seq=1 ttl=63 time=217.225 ms
    64 bytes from 10.0.255.2: icmp_seq=2 ttl=63 time=120.648 ms
    64 bytes from 10.0.255.2: icmp_seq=3 ttl=63 time=122.597 ms

    --- 10.0.255.2 ping statistics ---
    4 packets transmitted, 4 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 120.648/169.376/217.225/47.758 ms

    root@vQFX-RE-Leaf1> ping 10.0.255.3    
    PING 10.0.255.3 (10.0.255.3): 56 data bytes
    64 bytes from 10.0.255.3: icmp_seq=0 ttl=63 time=146.134 ms
    64 bytes from 10.0.255.3: icmp_seq=1 ttl=63 time=132.119 ms
    64 bytes from 10.0.255.3: icmp_seq=2 ttl=63 time=123.679 ms
    64 bytes from 10.0.255.3: icmp_seq=3 ttl=63 time=118.110 ms
    ^C
    --- 10.0.255.3 ping statistics ---
    4 packets transmitted, 4 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 118.110/130.011/146.134/10.561 ms

*PS: p2p линки тоже пингуются 100ms+, вероятно какая-то проблема виртаулизации*

 Добавленна балансировка 

set policy-options policy-statement LB term LB then load-balance per-packet (На всякий случай. Даная конфа = Per-flow балансировке) \
set routing-options forwarding-table export LB


    root@vQFX-RE-Leaf1> show route forwarding-table destination 10.0.255.2 
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

#### Таблица маршрутизации

root@vQFX-RE-Leaf1> show route 

    inet.0: 15 destinations, 15 routes (15 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    10.0.0.1/32        *[IS-IS/18] 00:07:46, metric 1
                        >  to 10.0.1.1 via xe-0/0/1.0
    10.0.0.2/32        *[IS-IS/18] 00:07:39, metric 1
                        >  to 10.0.2.1 via xe-0/0/2.0
    10.0.1.0/31        *[Direct/0] 3d 03:07:47
                        >  via xe-0/0/1.0
    10.0.1.0/32        *[Local/0] 3d 03:07:47
                           Local via xe-0/0/1.0
    10.0.1.2/31        *[IS-IS/18] 00:07:46, metric 2
                        >  to 10.0.1.1 via xe-0/0/1.0
    10.0.1.4/31        *[IS-IS/18] 00:07:46, metric 2
                        >  to 10.0.1.1 via xe-0/0/1.0
    10.0.2.0/31        *[Direct/0] 1w2d 19:44:07
                        >  via xe-0/0/2.0
    10.0.2.0/32        *[Local/0] 1w2d 19:44:07
                           Local via xe-0/0/2.0
    10.0.2.2/31        *[IS-IS/18] 00:07:39, metric 2
                        >  to 10.0.2.1 via xe-0/0/2.0
    10.0.2.4/31        *[IS-IS/18] 00:07:39, metric 2
                        >  to 10.0.2.1 via xe-0/0/2.0
    10.0.255.1/32      *[Direct/0] 1w2d 19:46:33
                        >  via lo0.0
    10.0.255.2/32      *[IS-IS/18] 00:03:34, metric 2
                        >  to 10.0.1.1 via xe-0/0/1.0
                           to 10.0.2.1 via xe-0/0/2.0
    10.0.255.3/32      *[IS-IS/18] 00:03:34, metric 2
                        >  to 10.0.1.1 via xe-0/0/1.0
                           to 10.0.2.1 via xe-0/0/2.0
    169.254.0.0/24     *[Direct/0] 1w2d 19:46:35
                        >  via em1.0
    169.254.0.2/32     *[Local/0] 1w2d 19:46:35
                           Local via em1.0

    iso.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    49.0001.0010.0025.5001/72                
                       *[Direct/0] 02:04:59
                        >  via lo0.0    

## Выполнение Cisco

Добавление конфигурации IS-IS  
Для всех Leaf одинаковая (Кроме NET идентификатора):
 + feature isis

 +  router isis underlay  
    net 49.0001.0010.0025.5003.00  
    is-type level-2  
    metric-style transition  

 +  interface loopback0  
    ip router isis underlay  
    isis passive-interface level-2  

    interface Ethernet1/1  
    ip router isis underlay  
    medium p2p  

    interface Ethernet1/2  
    ip router isis underlay  
    medium p2p

Для всех Spine одинаковая (Кроме NET идентификатора):

 + feature isis

 +  router isis underlay  
    net 49.0001.0010.0025.5003.00  
    is-type level-2  
    metric-style transition  

 +  interface loopback0  
    ip router isis underlay  
    isis passive-interface level-2  

    interface Ethernet1/1  
    ip router isis underlay  
    medium p2p  

    interface Ethernet1/2  
    ip router isis underlay  
    medium p2p

    interface Ethernet1/2  
    ip router isis underlay  
    medium p2p  

Сверка database

NXOS-**Leaf1**# sh isis database

    IS-IS Process: underlay LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      NXOS-Spine1.00-00     0x00000009   0x5502    1105       0/0/0/3
      NXOS-Spine2.00-00     0x00000008   0xFC49    1128       0/0/0/3
      NXOS-Leaf1.00-00    * 0x0000000B   0xA404    1127       0/0/0/3
      NXOS-Leaf2.00-00      0x00000008   0x6035    1136       0/0/0/3
      NXOS-Leaf3.00-00      0x00000008   0x6D12    1180       0/0/0/3


NXOS-Leaf2# sh isis database

    IS-IS Process: underlay LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      NXOS-Spine1.00-00     0x00000009   0x5502    1105       0/0/0/3
      NXOS-Spine2.00-00     0x00000008   0xFC49    1128       0/0/0/3
      NXOS-Leaf1.00-00      0x0000000B   0xA404    1125       0/0/0/3
      NXOS-Leaf2.00-00    * 0x00000008   0x6035    1138       0/0/0/3
      NXOS-Leaf3.00-00      0x00000008   0x6D12    1180       0/0/0/3


NXOS-Leaf3# sh isis database

    IS-IS Process: underlay LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      NXOS-Spine1.00-00     0x00000009   0x5502    1105       0/0/0/3
      NXOS-Spine2.00-00     0x00000008   0xFC49    1128       0/0/0/3
      NXOS-Leaf1.00-00      0x0000000B   0xA404    1125       0/0/0/3
      NXOS-Leaf2.00-00      0x00000008   0x6035    1136       0/0/0/3
      NXOS-Leaf3.00-00    * 0x00000008   0x6D12    1182       0/0/0/3


NXOS-Spine1# sh isis database

    IS-IS Process: underlay LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      NXOS-Spine1.00-00   * 0x00000009   0x5502    1106       0/0/0/3
      NXOS-Spine2.00-00     0x00000008   0xFC49    1127       0/0/0/3
      NXOS-Leaf1.00-00      0x0000000B   0xA404    1126       0/0/0/3
      NXOS-Leaf2.00-00      0x00000008   0x6035    1137       0/0/0/3
      NXOS-Leaf3.00-00      0x00000008   0x6D12    1181       0/0/0/3 

NXOS-Spine2# sh isis database

    IS-IS Process: underlay LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      NXOS-Spine1.00-00     0x00000009   0x5502    1104       0/0/0/3
      NXOS-Spine2.00-00   * 0x00000008   0xFC49    1129       0/0/0/3
      NXOS-Leaf1.00-00      0x0000000B   0xA404    1126       0/0/0/3
      NXOS-Leaf2.00-00      0x00000008   0x6035    1137       0/0/0/3
      NXOS-Leaf3.00-00      0x00000008   0x6D12    1181       0/0/0/3


Пинг Leaf 1 -> Leaf 2/3

    NXOS-Leaf1# ping 10.0.255.2
    PING 10.0.255.2 (10.0.255.2): 56 data bytes
    64 bytes from 10.0.255.2: icmp_seq=0 ttl=253 time=41.513 ms
    64 bytes from 10.0.255.2: icmp_seq=1 ttl=253 time=8.108 ms
    64 bytes from 10.0.255.2: icmp_seq=2 ttl=253 time=7.044 ms
    64 bytes from 10.0.255.2: icmp_seq=3 ttl=253 time=6.695 ms
    64 bytes from 10.0.255.2: icmp_seq=4 ttl=253 time=7.301 ms

    --- 10.0.255.2 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 6.695/14.132/41.513 ms

    NXOS-Leaf1# ping 10.0.255.3
    PING 10.0.255.3 (10.0.255.3): 56 data bytes
    64 bytes from 10.0.255.3: icmp_seq=0 ttl=253 time=23.27 ms
    64 bytes from 10.0.255.3: icmp_seq=1 ttl=253 time=10.128 ms
    64 bytes from 10.0.255.3: icmp_seq=2 ttl=253 time=8.536 ms
    64 bytes from 10.0.255.3: icmp_seq=3 ttl=253 time=7.842 ms
    64 bytes from 10.0.255.3: icmp_seq=4 ttl=253 time=8.393 ms

    --- 10.0.255.3 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 7.842/11.633/23.27 ms


ECMP на NEXUS включён по дефолту

    NXOS-Leaf1# sh forwarding 10.0.255.1 detail        

    Prefix 10.0.255.2/32, No of paths: 2, Update time: Sat Dec 20 10:38:02 2025
       Partial Install: No
       10.0.1.1                                  Ethernet1/1         
       10.0.2.1                                  Ethernet1/2

#### Таблица маршрутизации

    NXOS-Leaf1# sh ip route 
    IP Route Table for VRF "default"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>

    10.0.0.1/32, ubest/mbest: 1/0
        *via 10.0.1.1, Eth1/1, [115/41], 00:21:48, isis-underlay, L2
    10.0.0.2/32, ubest/mbest: 1/0
        *via 10.0.2.1, Eth1/2, [115/41], 00:18:20, isis-underlay, L2
    10.0.1.0/31, ubest/mbest: 1/0, attached
        *via 10.0.1.0, Eth1/1, [0/0], 1w2d, direct
    10.0.1.0/32, ubest/mbest: 1/0, attached
        *via 10.0.1.0, Eth1/1, [0/0], 1w2d, local
    10.0.1.2/31, ubest/mbest: 1/0
        *via 10.0.1.1, Eth1/1, [115/80], 00:21:48, isis-underlay, L2
    10.0.1.4/31, ubest/mbest: 1/0
        *via 10.0.1.1, Eth1/1, [115/80], 00:21:48, isis-underlay, L2
    10.0.2.0/31, ubest/mbest: 1/0, attached
        *via 10.0.2.0, Eth1/2, [0/0], 1w2d, direct
    10.0.2.0/32, ubest/mbest: 1/0, attached
        *via 10.0.2.0, Eth1/2, [0/0], 1w2d, local
    10.0.2.2/31, ubest/mbest: 1/0
        *via 10.0.2.1, Eth1/2, [115/80], 00:18:20, isis-underlay, L2
    10.0.2.4/31, ubest/mbest: 1/0
        *via 10.0.2.1, Eth1/2, [115/80], 00:18:20, isis-underlay, L2
    10.0.255.1/32, ubest/mbest: 2/0, attached
        *via 10.0.255.1, Lo0, [0/0], 1w2d, local
        *via 10.0.255.1, Lo0, [0/0], 1w2d, direct
    10.0.255.2/32, ubest/mbest: 2/0
        *via 10.0.1.1, Eth1/1, [115/81], 00:20:37, isis-underlay, L2
        *via 10.0.2.1, Eth1/2, [115/81], 00:18:11, isis-underlay, L2
    10.0.255.3/32, ubest/mbest: 2/0
        *via 10.0.1.1, Eth1/1, [115/81], 00:18:50, isis-underlay, L2
        *via 10.0.2.1, Eth1/2, [115/81], 00:18:20, isis-underlay, L2