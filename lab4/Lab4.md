# Д/з №3
## Построение Underlay сети(eBGP) 
Цель:
Настроить BGP для Underlay сети.

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
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab4/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0.png)

## Сводная таблица AS
|  Оборудование  |      Lo0      |   AS  |
| -------------- |---------------|-------|
|    Leaf1       | 10.0.255.1/32 | 65001 |
|    Leaf2       | 10.0.255.2/32 | 65002 |
|    Leaf3       | 10.0.255.3/32 | 65003 |
|    Spine1      | 10.0.0.1/32   | 65000 |
|    Spine2      | 10.0.0.2/32   | 65000 |


## Выполнение Juniper
Добавление конфигурации BGP  
Для всех Leaf одинаковая (Кроме autonomous-system и ip neighbor):

#### Общие настройки BGP, BFD

+ set routing-options autonomous-system 65001  
set protocols bgp group SPINE log-updown  
set protocols bgp group SPINE export Export_Loopback_BGP  
set protocols bgp group SPINE graceful-restart  
set protocols bgp group SPINE multipath multiple-as  
set protocols bgp group SPINE bfd-liveness-detection minimum-interval 300  
set protocols bgp group SPINE bfd-liveness-detection multiplier 3  
set protocols bgp group SPINE bfd-liveness-detection session-mode single-hop  
set protocols bgp group SPINE neighbor 10.0.1.1 peer-as 65000  
set protocols bgp group SPINE neighbor 10.0.2.1 peer-as 65000  

#### Политика для распространения lo 
+ set policy-options policy-statement Export_Loopback_BGP term 1 from protocol direct  
set policy-options policy-statement Export_Loopback_BGP term 1 from route-filter 10.0.255.0/24 prefix-length-range /32-/32  
set policy-options policy-statement Export_Loopback_BGP term 1 then accept  
set protocols bgp group SPINE export Export_Loopback_BGP  

#### Политика для балансировки
+ set policy-options policy-statement LB term LB then load-balance per-packet  
set routing-options forwarding-table export LB  
  
    
Для всех Spine одинаковая (Кроме autonomous-system и ip neighbor):


#### Общие настройки BGP, BFD

+ set routing-options autonomous-system 65000  
set protocols bgp group LEAF log-updown  
set protocols bgp group LEAF export Export_Loopback_BGP  
set protocols bgp group LEAF graceful-restart  
set protocols bgp group LEAF multipath multiple-as  
set protocols bgp group LEAF bfd-liveness-detection minimum-interval 300  
set protocols bgp group LEAF bfd-liveness-detection multiplier 3  
set protocols bgp group LEAF bfd-liveness-detection session-mode single-hop  
set protocols bgp group LEAF neighbor 10.0.1.0 peer-as 65001  
set protocols bgp group LEAF neighbor 10.0.1.2 peer-as 65002  
set protocols bgp group LEAF neighbor 10.0.1.4 peer-as 65003  

#### Политика для распространения lo 

+ set policy-options policy-statement Export_Loopback_BGP term 1 from protocol direct  
set policy-options policy-statement Export_Loopback_BGP term 1 from route-filter 10.0.0.0/24 prefix-length-range /32-/32  
set policy-options policy-statement Export_Loopback_BGP term 1 then accept  
set protocols bgp group LEAF export Export_Loopback_BGP  

#### Политика для балансировки

+ set policy-options policy-statement LB term LB then load-balance per-packet  
set routing-options forwarding-table export LB  

### Состояние BGP, BFD

root@vQFX-RE-**Leaf1**> show bgp summary   

      Threading mode: BGP I/O  
      Groups: 1 Peers: 2 Down peers: 0  
        Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending  
        inet.0                 
                           6          6          0          0          0          0  
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...  
    10.0.1.1              65000        401        402       0       1     2:57:41 Establ  
      inet.0: 3/3/3/0  
    10.0.2.1              65000        149        147       0       2     1:05:18 Establ  
      inet.0: 3/3/3/0  


root@vQFX-RE-**Leaf1**> show bfd session brief   

                                                      Detect   Transmit  
    Address                  State     Interface      Time     Interval  Multiplier  
    10.0.1.1                 Up        xe-0/0/1.0     0.900     0.300        3     
    10.0.2.1                 Up        xe-0/0/2.0     0.900     0.300        3     

    2 sessions, 2 clients  
    Cumulative transmit rate 6.7 pps, cumulative receive rate 6.7 pps  



root@vQFX-RE-**Leaf2**> show bgp summary 

    Threading mode: BGP I/O
    Groups: 1 Peers: 2 Down peers: 0
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                           6          6          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.1.3              65000        407        401       0       0     2:59:29 Establ
      inet.0: 3/3/3/0
    10.0.2.3              65000        388        384       0       0     2:52:08 Establ
      inet.0: 3/3/3/0

root@vQFX-RE-**Leaf2**> show bfd session brief 

                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    10.0.1.3                 Up        xe-0/0/1.0     0.900     0.300        3   
    10.0.2.3                 Up        xe-0/0/2.0     0.900     0.300        3   

    2 sessions, 2 clients
    Cumulative transmit rate 6.7 pps, cumulative receive rate 6.7 pps

root@vQFX-RE-**Leaf3**> show bgp summary  

    Threading mode: BGP I/O
    Groups: 1 Peers: 2 Down peers: 0
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                           6          6          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.1.5              65000        143        144       0       3     1:03:33 Establ
      inet.0: 3/3/3/0
    10.0.2.5              65000        392        389       0       0     2:53:27 Establ
      inet.0: 3/3/3/0


root@vQFX-RE-**Leaf3**> show bfd session brief  

                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    10.0.1.5                 Up        xe-0/0/1.0     0.900     0.300        3   
    10.0.2.5                 Up        xe-0/0/2.0     0.900     0.300        3   

    2 sessions, 2 clients
    Cumulative transmit rate 6.7 pps, cumulative receive rate 6.7 pps


root@vQFX-RE-**Spine1**> show bgp summary 

    Threading mode: BGP I/O
    Groups: 1 Peers: 3 Down peers: 0
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                           3          3          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.1.0              65001        426        423       0       1     3:08:10 Establ
      inet.0: 1/1/1/0
    10.0.1.2              65002        412        417       0       0     3:04:11 Establ
      inet.0: 1/1/1/0
    10.0.1.4              65003        152        149       0       3     1:06:49 Establ
      inet.0: 1/1/1/0

root@vQFX-RE-**Spine1**> show bfd session brief 

                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    10.0.1.0                 Up        xe-0/0/1.0     0.900     0.300        3   
    10.0.1.2                 Up        xe-0/0/2.0     0.900     0.300        3   
    10.0.1.4                 Up        xe-0/0/3.0     0.900     0.300        3   

    3 sessions, 3 clients
    Cumulative transmit rate 10.0 pps, cumulative receive rate 10.0 pps


root@vQFX-RE-**Spine2**> show bgp summary 

    Threading mode: BGP I/O
    Groups: 1 Peers: 3 Down peers: 0
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                           3          3          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.2.0              65001        175        174       0       2     1:17:28 Establ
      inet.0: 1/1/1/0
    10.0.2.2              65002        400        402       0       0     2:58:33 Establ
      inet.0: 1/1/1/0
    10.0.2.4              65003        400        401       0       0     2:58:24 Establ
      inet.0: 1/1/1/0

root@vQFX-RE-**Spine2**> show bfd session brief 

                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    10.0.2.0                 Up        xe-0/0/1.0     0.900     0.300        3   
    10.0.2.2                 Up        xe-0/0/2.0     0.900     0.300        3   
    10.0.2.4                 Up        xe-0/0/3.0     0.900     0.300        3   

    3 sessions, 3 clients
    Cumulative transmit rate 10.0 pps, cumulative receive rate 10.0 pps

Пинг Leaf 1 -> Leaf 2/3 

root@vQFX-RE-Leaf1> ping 10.0.255.2  

    PING 10.0.255.2 (10.0.255.2): 56 data bytes
    64 bytes from 10.0.255.2: icmp_seq=0 ttl=63 time=400.660 ms
    64 bytes from 10.0.255.2: icmp_seq=1 ttl=63 time=314.399 ms
    64 bytes from 10.0.255.2: icmp_seq=2 ttl=63 time=439.652 ms
    64 bytes from 10.0.255.2: icmp_seq=3 ttl=63 time=250.332 ms
    64 bytes from 10.0.255.2: icmp_seq=4 ttl=63 time=478.673 ms
    ^C
    --- 10.0.255.2 ping statistics ---
    5 packets transmitted, 5 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 250.332/376.743/478.673/83.410 ms

root@vQFX-RE-Leaf1> ping 10.0.255.3    

    PING 10.0.255.3 (10.0.255.3): 56 data bytes
    64 bytes from 10.0.255.3: icmp_seq=0 ttl=63 time=399.397 ms
    64 bytes from 10.0.255.3: icmp_seq=1 ttl=63 time=240.649 ms
    64 bytes from 10.0.255.3: icmp_seq=2 ttl=63 time=356.099 ms
    64 bytes from 10.0.255.3: icmp_seq=3 ttl=63 time=354.528 ms
    ^C
    --- 10.0.255.3 ping statistics ---
    4 packets transmitted, 4 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 240.649/337.668/399.397/58.837 ms

### Таблица маршрутизации

root@vQFX-RE-Leaf1> show route 

    inet.0: 11 destinations, 13 routes (11 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    10.0.0.1/32        *[BGP/170] 02:50:52, localpref 100
                          AS path: 65000 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
    10.0.0.2/32        *[BGP/170] 01:21:33, localpref 100
                          AS path: 65000 I, validation-state: unverified
                        >  to 10.0.2.1 via xe-0/0/2.0
    10.0.1.0/31        *[Direct/0] 3w5d 03:31:14
                        >  via xe-0/0/1.0
    10.0.1.0/32        *[Local/0] 3w5d 03:31:14
                           Local via xe-0/0/1.0
    10.0.2.0/31        *[Direct/0] 4w4d 20:07:34
                        >  via xe-0/0/2.0
    10.0.2.0/32        *[Local/0] 4w4d 20:07:34
                           Local via xe-0/0/2.0
    10.0.255.1/32      *[Direct/0] 02:21:00
                        >  via lo0.0
    10.0.255.2/32      *[BGP/170] 01:21:33, localpref 100
                          AS path: 65000 65002 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                           to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 01:21:33, localpref 100
                          AS path: 65000 65002 I, validation-state: unverified
                        >  to 10.0.2.1 via xe-0/0/2.0
    10.0.255.3/32      *[BGP/170] 01:12:35, localpref 100, from 10.0.2.1
                          AS path: 65000 65003 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                           to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 01:12:35, localpref 100
                          AS path: 65000 65003 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
    169.254.0.0/24     *[Direct/0] 4w4d 20:10:02
                        >  via em1.0
    169.254.0.2/32     *[Local/0] 4w4d 20:10:02
                           Local via em1.0

Балансировка работает

root@vQFX-RE-Leaf1> show route forwarding-table 

    Routing table: default.inet
    Internet:
    Enabled protocols: Bridging, 
    Destination        Type RtRef Next hop           Type Index    NhRef Netif
    default            perm     0                    rjct       51     1
    0.0.0.0/32         perm     0                    dscd       49     1
    10.0.0.1/32        user     0 10.0.1.1           ucst     1734     5 xe-0/0/1.0
    10.0.0.2/32        user     0 10.0.2.1           ucst     1735     5 xe-0/0/2.0
    10.0.1.0/31        intf     0                    rslv     1731     1 xe-0/0/1.0
    10.0.1.0/32        intf     0 10.0.1.0           locl     1730     2
    10.0.1.0/32        dest     0 10.0.1.0           locl     1730     2
    10.0.1.1/32        dest     1 2:5:86:71:cd:7     ucst     1734     5 xe-0/0/1.0
    10.0.2.0/31        intf     0                    rslv     1733     1 xe-0/0/2.0
    10.0.2.0/32        intf     0 10.0.2.0           locl     1732     2
    10.0.2.0/32        dest     0 10.0.2.0           locl     1732     2
    10.0.2.1/32        dest     1 2:5:86:71:2d:7     ucst     1735     5 xe-0/0/2.0
    10.0.255.1/32      intf     0 10.0.255.1         locl     1700     1
    10.0.255.2/32      user     0                    ulst   131070     3
                                  10.0.1.1           ucst     1734     5 xe-0/0/1.0
                                  10.0.2.1           ucst     1735     5 xe-0/0/2.0
    10.0.255.3/32      user     0                    ulst   131070     3
                                  10.0.1.1           ucst     1734     5 xe-0/0/1.0
                                  10.0.2.1           ucst     1735     5 xe-0/0/2.0
    169.254.0.0/24     intf     0                    rslv      323     1 em1.0
    169.254.0.0/32     dest     0 169.254.0.0        recv      321     1 em1.0
    169.254.0.1/32     dest     1 50:8e:bc:4:74:1    ucst      340     2 em1.0
    169.254.0.2/32     intf     0 169.254.0.2        locl      322     2
    169.254.0.2/32     dest     0 169.254.0.2        locl      322     2
    169.254.0.255/32   dest     0 169.254.0.255      bcst      320     1 em1.0
    224.0.0.0/4        perm     0                    mdsc       50     1
    224.0.0.1/32       perm     0 224.0.0.1          mcst       46     1
    255.255.255.255/32 perm     0                    bcst       47     1    

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