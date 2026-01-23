# Д/з №4
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
Добавление конфигурации BGP, BFD  
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

    inet.0: 9 destinations, 11 routes (9 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    10.0.1.0/31        *[Direct/0] 3w6d 09:46:30
                        >  via xe-0/0/1.0
    10.0.1.0/32        *[Local/0] 3w6d 09:46:30
                           Local via xe-0/0/1.0
    10.0.2.0/31        *[Direct/0] 4w6d 02:22:50
                        >  via xe-0/0/2.0
    10.0.2.0/32        *[Local/0] 4w6d 02:22:50
                           Local via xe-0/0/2.0
    10.0.255.1/32      *[Direct/0] 1d 08:36:16
                        >  via lo0.0
    10.0.255.2/32      *[BGP/170] 02:17:58, localpref 100, from 10.0.2.1
                          AS path: 65000 65002 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                           to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:17:58, localpref 100
                          AS path: 65000 65002 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
    10.0.255.3/32      *[BGP/170] 02:16:33, localpref 100
                          AS path: 65000 65003 I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                           to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:16:33, localpref 100
                          AS path: 65000 65003 I, validation-state: unverified

Балансировка работает

root@vQFX-RE-Leaf1> show route forwarding-table 

    Routing table: default.inet
    Internet:
    Enabled protocols: Bridging, 
    Destination        Type RtRef Next hop           Type Index    NhRef Netif
    default            perm     0                    rjct       51     1
    0.0.0.0/32         perm     0                    dscd       49     1
    10.0.1.0/31        intf     0                    rslv     1731     1 xe-0/0/1.0
    10.0.1.0/32        intf     0 10.0.1.0           locl     1730     2
    10.0.1.0/32        dest     0 10.0.1.0           locl     1730     2
    10.0.1.1/32        dest     1 2:5:86:71:cd:7     ucst     1734     4 xe-0/0/1.0
    10.0.2.0/31        intf     0                    rslv     1733     1 xe-0/0/2.0
    10.0.2.0/32        intf     0 10.0.2.0           locl     1732     2
    10.0.2.0/32        dest     0 10.0.2.0           locl     1732     2
    10.0.2.1/32        dest     1 2:5:86:71:2d:7     ucst     1735     4 xe-0/0/2.0
    10.0.255.1/32      intf     0 10.0.255.1         locl     1700     1
    10.0.255.2/32      user     0                    ulst   131070     3
                                  10.0.1.1           ucst     1734     4 xe-0/0/1.0
                                  10.0.2.1           ucst     1735     4 xe-0/0/2.0
    10.0.255.3/32      user     0                    ulst   131070     3
                                  10.0.1.1           ucst     1734     4 xe-0/0/1.0
                                  10.0.2.1           ucst     1735     4 xe-0/0/2.0
    169.254.0.0/24     intf     0                    rslv      323     1 em1.0
    169.254.0.0/32     dest     0 169.254.0.0        recv      321     1 em1.0
    169.254.0.1/32     dest     1 50:8e:bc:4:74:1    ucst      340     2 em1.0
    169.254.0.2/32     intf     0 169.254.0.2        locl      322     2
    169.254.0.2/32     dest     0 169.254.0.2        locl      322     2
    169.254.0.255/32   dest     0 169.254.0.255      bcst      320     1 em1.0
    224.0.0.0/4        perm     0                    mdsc       50     1
    224.0.0.1/32       perm     0 224.0.0.1          mcst       46     1
    255.255.255.255/32 perm     0                    bcst       47     

## Выполнение Cisco

Добавление конфигурации BGP, BFD
Для всех Leaf одинаковая (Кроме autonomous-system и ip neighbor)

+ feature bgp  
+ router bgp 65001  
   + router-id 10.0.255.1  
  reconnect-interval 12  
  log-neighbor-changes  
  address-family ipv4 unicast  
     + redistribute direct route-map EXPORT-Loopback  
       maximum-paths 2  
  + template peer SPINE  
     + bfd  
    remote-as 65000  
    timers 3 9  
    address-family ipv4 unicast  
  + neighbor 10.0.1.1  
     + inherit peer SPINE  
  + neighbor 10.0.2.1  
     + inherit peer SPINE 

+ feature bfd  
bfd interval 300 min_rx 300 multiplier 3 

#### Политика для распространения lo 

+ route-map EXPORT-Loopback permit 10  
  match ip address prefix-list Looback_for_BGP 

+ ip prefix-list Looback_for_BGP seq 5 permit 10.0.255.0/24 eq 32 


Для всех Spine одинаковая (Кроме autonomous-system и ip neighbor):

#### Общие настройки BGP, BFD


+ feature bgp

+ router bgp 65000  
  + router-id 10.0.0.1  
  reconnect-interval 12  
  log-neighbor-changes  
  address-family ipv4 unicast  
    + maximum-paths 2  
  + neighbor 10.0.0.0/22 remote-as route-map AS-LEAF  
    bfd  
    address-family ipv4 unicast  
+ route-map AS-LEAF permit 10  
   match as-number 65001-65100 

+ feature bfd  
bfd interval 300 min_rx 300 multiplier 3

### Состояние BGP, BFD

NXOS-**Leaf1** sh bgp  sessions 

    Total peers 2, established peers 2
    ASN 65001
    VRF default, local ASN 65001
    peers 2, established peers 2, local router-id 10.0.255.1
    State: I-Idle, A-Active, O-Open, E-Established, C-Closing, S-Shutdown

    Neighbor        ASN    Flaps LastUpDn|LastRead|LastWrit St Port(L/R)  Notif(S/R)
    10.0.1.1        65000 2     1d02h   |00:00:01|00:00:01 E   44079/179        2/0
    10.0.2.1        65000 0     1d03h   |0.622182|00:00:01 E   47041/179        0/0   


NXOS-**Leaf2**# sh bgp  sessions 

    Total peers 2, established peers 2
    ASN 65002
    VRF default, local ASN 65002
    peers 2, established peers 2, local router-id 10.0.255.2
    State: I-Idle, A-Active, O-Open, E-Established, C-Closing, S-Shutdown

    Neighbor        ASN    Flaps LastUpDn|LastRead|LastWrit St Port(L/R)  Notif(S/R)
    10.0.1.3        65000 0     1d03h   |0.511890|00:00:02 E   41254/179        0/0
    10.0.2.3        65000 0     1d03h   |00:00:02|00:00:02 E   16702/179        0/0


NXOS-**Leaf3**# sh bgp sessions 

    Total peers 2, established peers 2
    ASN 65003
    VRF default, local ASN 65003
    peers 2, established peers 2, local router-id 10.0.255.3
    State: I-Idle, A-Active, O-Open, E-Established, C-Closing, S-Shutdown

    Neighbor        ASN    Flaps LastUpDn|LastRead|LastWrit St Port(L/R)  Notif(S/R)
    10.0.1.5        65000 0     1d03h   |00:00:01|0.067959 E   28599/179        0/0
    10.0.2.5        65000 0     1d03h   |0.008743|0.069454 E   39301/179        0/0


NXOS-**Spine1**# sh bgp sessions 

    Total peers 3, established peers 3
    ASN 65000
    VRF default, local ASN 65000
    peers 3, established peers 3, local router-id 10.0.0.1
    State: I-Idle, A-Active, O-Open, E-Established, C-Closing, S-Shutdown

    Neighbor        ASN    Flaps LastUpDn|LastRead|LastWrit St Port(L/R)  Notif(S/R)
    10.0.1.0        65001 0     1d02h   |00:00:02|0.322892 E   179/44079      0/0
    10.0.1.2        65002 0     1d03h   |00:00:02|0.322992 E   179/41254      0/0
    10.0.1.4        65003 0     1d03h   |00:00:02|0.323061 E   179/28599      0/0

NXOS-**Spine2**# sh bgp  sessions 

    Total peers 3, established peers 3
    ASN 65000
    VRF default, local ASN 65000
    peers 3, established peers 3, local router-id 10.0.0.2
    State: I-Idle, A-Active, O-Open, E-Established, C-Closing, S-Shutdown

    Neighbor        ASN    Flaps LastUpDn|LastRead|LastWrit St Port(L/R)  Notif(S/R)
    10.0.2.0        65001 0     1d03h   |00:00:01|0.717002 E   179/47041      0/0
    10.0.2.2        65002 0     1d03h   |0.960260|0.717128 E   179/16702      0/0
    10.0.2.4        65003 0     1d03h   |0.774557|0.717182 E   179/39301      0/0

_К сожалению поднять bfd на cisco не вышло, после перебора всех возможных видов конфигураций и отсутвие каких-либо пакетов в дампе, сделал вывод, что проблема в виртуализации._

NXOS-Spine1# sh bfd neighbors 

    OurAddr         NeighAddr       LD/RD                 RH/RS           Holdown(mu
    lt)     State       Int                   Vrf                              Type 

    10.0.1.3        10.0.1.2        1090519045/0          Down            N/A(3)    
            Down        Eth1/2                default                          SH   

    10.0.1.5        10.0.1.4        1090519046/0          Down            N/A(3)    
            Down        Eth1/3                default                          SH   

    10.0.1.1        10.0.1.0        1090519049/0          Down            N/A(3)    
            Down        Eth1/1                default                          SH   


Пинг Leaf 1 -> Leaf 2/3

NXOS-Leaf1# ping 10.0.255.2 source 10.0.255.1

    PING 10.0.255.2 (10.0.255.2) from 10.0.255.1: 56 data bytes
    64 bytes from 10.0.255.2: icmp_seq=0 ttl=253 time=40.745 ms
    64 bytes from 10.0.255.2: icmp_seq=1 ttl=253 time=9.82 ms
    64 bytes from 10.0.255.2: icmp_seq=2 ttl=253 time=9.552 ms
    64 bytes from 10.0.255.2: icmp_seq=3 ttl=253 time=6.972 ms
    64 bytes from 10.0.255.2: icmp_seq=4 ttl=253 time=8.444 ms

    --- 10.0.255.2 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 6.972/15.106/40.745 ms

NXOS-Leaf1# ping 10.0.255.3 source 10.0.255.1

    PING 10.0.255.3 (10.0.255.3) from 10.0.255.1: 56 data bytes
    64 bytes from 10.0.255.3: icmp_seq=0 ttl=253 time=15.116 ms
    64 bytes from 10.0.255.3: icmp_seq=1 ttl=253 time=7.173 ms
    64 bytes from 10.0.255.3: icmp_seq=2 ttl=253 time=10.451 ms
    64 bytes from 10.0.255.3: icmp_seq=3 ttl=253 time=8.42 ms
    64 bytes from 10.0.255.3: icmp_seq=4 ttl=253 time=10.286 ms

    --- 10.0.255.3 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 7.173/10.289/15.116 ms


ECMP на NEXUS включён по дефолту

NXOS-Leaf1# sh forwarding 10.0.255.1 detail        

    Prefix 10.0.255.2/32, No of paths: 2, Update time: Mon Jan 12 13:03:18 2026
       Vobj id: 8   Partial Install: No
       10.0.1.1                                  Ethernet1/1         
       10.0.2.1                                  Ethernet1/2         

#### Таблица маршрутизации

NXOS-Leaf1# sh ip route 

    IP Route Table for VRF "default"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>

    10.0.1.0/31, ubest/mbest: 1/0, attached
        *via 10.0.1.0, Eth1/1, [0/0], 1d02h, direct
    10.0.1.0/32, ubest/mbest: 1/0, attached
        *via 10.0.1.0, Eth1/1, [0/0], 1d02h, local
    10.0.2.0/31, ubest/mbest: 1/0, attached
        *via 10.0.2.0, Eth1/2, [0/0], 3d00h, direct
    10.0.2.0/32, ubest/mbest: 1/0, attached
        *via 10.0.2.0, Eth1/2, [0/0], 3d00h, local
    10.0.255.1/32, ubest/mbest: 2/0, attached
        *via 10.0.255.1, Lo0, [0/0], 4w6d, local
        *via 10.0.255.1, Lo0, [0/0], 4w6d, direct
    10.0.255.2/32, ubest/mbest: 2/0
        *via 10.0.1.1, [20/0], 1d02h, bgp-65001, external, tag 65000
        *via 10.0.2.1, [20/0], 1d03h, bgp-65001, external, tag 65000
    10.0.255.3/32, ubest/mbest: 2/0
        *via 10.0.1.1, [20/0], 1d02h, bgp-65001, external, tag 65000
        *via 10.0.2.1, [20/0], 1d03h, bgp-65001, external, tag 65000
