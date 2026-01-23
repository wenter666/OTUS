# Д/з №3
## Построение Underlay сети(eBGP) 
Цель:
Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

## Сводная таблица (Добавлена адресация Client1-4)
|  Оборудование  |      Lo0       |
| -------------- |----------------| 
|    Leaf1       | 10.0.255.1/32  | 
|    Leaf2       | 10.0.255.2/32  | 
|    Leaf3       | 10.0.255.3/32  | 
|    Spine1      | 10.0.0.1/32    |
|    Spine2      | 10.0.0.2/32    |
|    Client1     | 192.168.0.1/32 | 
|    Client2     | 192.168.0.2/32 | 
|    Client3     | 192.168.0.3/32 |
|    Client4     | 192.168.0.4/32 |

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
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab5/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0.png)

## Сводная таблица AS
|  Оборудование  |      Lo0      |   AS  |
| -------------- |---------------|-------|
|    Leaf1       | 10.0.255.1/32 | 65001 |
|    Leaf2       | 10.0.255.2/32 | 65002 |
|    Leaf3       | 10.0.255.3/32 | 65003 |
|    Spine1      | 10.0.0.1/32   | 65000 |
|    Spine2      | 10.0.0.2/32   | 65000 |


## Выполнение Juniper
Добавление конфигурации BGP EVPN
Для всех Leaf одинаковая (Кроме autonomous-system, ip neighbor, RT/RD):

#### Общие настройки BGP EVPN, VxLAN

+ set routing-options autonomous-system 65001    
set routing-options router-id 10.0.255.1  
set routing-options autonomous-system 65000  ***Для IBGP***  


+ set protocols bgp group UNDERLAY_EBGP log-updown  
set protocols bgp group UNDERLAY_EBGP export Export_Loopback_BGP  
set protocols bgp group UNDERLAY_EBGP local-as 65001  
set protocols bgp group UNDERLAY_EBGP graceful-restart  
set protocols bgp group UNDERLAY_EBGP multipath multiple-as  
set protocols bgp group UNDERLAY_EBGP bfd-liveness-detection minimum-interval 300  
set protocols bgp group UNDERLAY_EBGP bfd-liveness-detection multiplier 3  
set protocols bgp group UNDERLAY_EBGP bfd-liveness-detection session-mode single-hop  
set protocols bgp group UNDERLAY_EBGP neighbor 10.0.1.1 peer-as 65100  
set protocols bgp group UNDERLAY_EBGP neighbor 10.0.2.1 peer-as 65200  

+ set protocols bgp group OVERLAY_EVPN-VxLAN type internal ***Для OVERLAY EBGP***  
set protocols bgp group OVERLAY_EVPN-VxLAN local-address 10.0.255.1  
set protocols bgp group OVERLAY_EVPN-VxLAN log-updown  
set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling  
set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart  
set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.1  
set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.2  

+ set protocols evpn encapsulation vxlan  
set protocols evpn extended-vni-list all - ***Для все VNI*** 

***Настройка в Default Switch, другой вариант испоользовать Routing Instance MAC-VRF***   
+ set switch-options vtep-source-interface lo0.0 - ***Соурс для туннелей***   
set switch-options route-distinguisher 10.0.255.1:65000 - ***Для уникальности MAC***  
set switch-options vrf-target target:65000:1 - ***target для EVPN type 1,5***  
set switch-options vrf-target auto - ***target для EVPN type 2,3***  
[Example: Configuring VNI Route Targets Automatically with Manual Override](https://www.juniper.net/documentation/us/en/software/junos/evpn/topics/example/vrf-target-auto-manual.html)

***Настройка vlan + привязка vni***
**LEAF1**  
+ set vlans vlan100 vlan-id 100  
set vlans vlan100 vxlan vni 100100  
set vlans vlan300 vlan-id 300  
set vlans vlan300 vxlan vni 300300  

**LEAF2**  
+ set vlans vlan100 vlan-id 100  
set vlans vlan100 vxlan vni 100100  

**LEAF3**  
+ set vlans vlan200 vlan-id 200   
set vlans vlan200 vxlan vni 300300 - ***Для примера где vlan разный но VNI один***   

#### Настройка интерфейсов однотипная
+ set interfaces xe-0/0/3 unit 0 family ethernet-switching interface-mode access  
set interfaces xe-0/0/3 unit 0 family ethernet-switching vlan members vlan200

#### Политика для распространения lo 
+ set policy-options policy-statement Export_Loopback_BGP term 1 from protocol direct  
set policy-options policy-statement Export_Loopback_BGP term 1 from route-filter 10.0.0.0/24 prefix-length-range /32-/32  
set policy-options policy-statement Export_Loopback_BGP term 1 then accept  

#### Политика для балансировки
+ set policy-options policy-statement LB term LB then load-balance per-packet  
set routing-options forwarding-table export LB  


Для всех Spine одинаковая (Кроме autonomous-system и ip neighbor):


#### Общие настройки BGP EVPN

+ set routing-options autonomous-system 65001  

+ set protocols bgp group UNDERLAY_EBGP log-updown  
set protocols bgp group UNDERLAY_EBGP export Export_Loopback_BGP  
set protocols bgp group UNDERLAY_EBGP local-as 65200  
set protocols bgp group UNDERLAY_EBGP graceful-restart  
set protocols bgp group UNDERLAY_EBGP multipath multiple-as  
set protocols bgp group UNDERLAY_EBGP bfd-liveness-detection minimum-interval 300  
set protocols bgp group UNDERLAY_EBGP bfd-liveness-detection multiplier 3  
set protocols bgp group UNDERLAY_EBGP bfd-liveness-detection session-mode single-hop  
set protocols bgp group UNDERLAY_EBGP neighbor 10.0.2.0 peer-as 65001  
set protocols bgp group UNDERLAY_EBGP neighbor 10.0.2.2 peer-as 65002  
set protocols bgp group UNDERLAY_EBGP neighbor 10.0.2.4 peer-as 65003  

+ set protocols bgp group OVERLAY_EVPN-VxLAN type internal  
set protocols bgp group OVERLAY_EVPN-VxLAN log-updown  
set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling  
set protocols bgp group OVERLAY_EVPN-VxLAN cluster 10.0.0.2 - ***RR***  
set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart    
set protocols bgp group OVERLAY_EVPN-VxLAN allow 10.0.255.0/24 - ***устнавливаются сессии с пирами в этой подесети*** 

#### Политика для распространения lo 
+ set policy-options policy-statement Export_Loopback_BGP term 1 from protocol direct  
set policy-options policy-statement Export_Loopback_BGP term 1 from route-filter 10.0.0.0/24 prefix-length-range /32-/32  
set policy-options policy-statement Export_Loopback_BGP term 1 then accept  

#### Политика для балансировки
+ set policy-options policy-statement LB term LB then load-balance per-packet  
set routing-options forwarding-table export LB  
  

### Состояние BGP (Только на SPINE)

root@vQFX-RE-**Spine1**> show bgp summary

    Threading mode: BGP I/O  
    Groups: 2 Peers: 6 Down peers: 0  
    Unconfigured peers: 3  
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending  
    inet.0                 
                          10          6          0          0          0          0  
    bgp.evpn.0             
                          16         16          0          0          0          0  
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...  
    10.0.1.0              65001        136        134       0      40       58:23 Establ  
      inet.0: 2/4/4/0  
    10.0.1.2              65002        331        332       0      51     2:26:36 Establ  
      inet.0: 2/3/3/0  
    10.0.1.4              65003        312        312       0      55     2:18:33 Establ  
      inet.0: 2/3/3/0  
    10.0.255.1            65000      22021      22002       0       0 6d 22:44:24 Establ  
      bgp.evpn.0: 7/7/7/0  
    10.0.255.2            65000      22000      22056       0       0 6d 22:42:57 Establ  
      bgp.evpn.0: 5/5/5/0  
    10.0.255.3            65000      22006      22045       0       0 6d 22:38:03 Establ  
      bgp.evpn.0: 4/4/4/0

root@vQFX-RE-****Spine2****> show bgp summary     

    Threading mode: BGP I/O
    Groups: 2 Peers: 6 Down peers: 0
    Unconfigured peers: 3
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                           8          6          0          0          0          0
    bgp.evpn.0           
                          16         16          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.2.0              65001        460        457       0      44     3:23:30 Establ
      inet.0: 2/2/2/0
    10.0.2.2              65002        269        268       0      40     1:58:39 Establ
      inet.0: 2/3/3/0
    10.0.2.4              65003        441        438       0      51     3:15:18 Establ
      inet.0: 2/3/3/0
    10.0.255.1            65000      22050      22034       0       0 6d 22:58:02 Establ
      bgp.evpn.0: 7/7/7/0
    10.0.255.2            65000      22016      22110       0       0 6d 22:50:11 Establ
      bgp.evpn.0: 5/5/5/0
    10.0.255.3            65000      22022      22100       0       0 6d 22:45:17 Establ
      bgp.evpn.0: 4/4/4/0

### Вывод по vlan

root@vQFX-RE-**Leaf1**> show vlans brief    

    Routing instance        VLAN name             Tag          Interfaces
    default-switch          default               1        
                                                                
    default-switch          vlan100               100      
                                                              vtep.32769*
                                                              xe-0/0/3.0*
    default-switch          vlan300               300      
                                                              vtep.32770*
                                                              xe-0/0/4.0*
root@vQFX-RE-**Leaf2**> show vlans brief 

    Routing instance        VLAN name             Tag          Interfaces
    default-switch          default               1        

    default-switch          vlan100               100      
                                                               vtep.32769*
                                                               xe-0/0/3.0*
root@vQFX-RE-**Leaf3**> show vlans brief     

    Routing instance        VLAN name             Tag          Interfaces
    default-switch          default               1        

    default-switch          vlan200               200      
                                                               vtep.32769*
                                                               xe-0/0/3.0*


#### Пинг Client1 ->  Cleint3/4 

root@cli:~# ping 192.168.0.3 

    PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
    64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=301 ms
    64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=510 ms
    64 bytes from 192.168.0.3: icmp_seq=3 ttl=64 time=340 ms
    ^C
    --- 192.168.0.3 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2000ms
    rtt min/avg/max/mdev = 301.645/384.132/510.282/90.598 ms

До Client 4 пинга есть, vlan и VNI совпадают.

root@cli:~# ping 192.168.0.4

    PING 192.168.0.4 (192.168.0.4) 56(84) bytes of data.
    ^C
    --- 192.168.0.4 ping statistics ---
    2 packets transmitted, 0 received, 100% packet loss, time 1026ms

До Client 3 пинга нет, как и должно быть VNI не совпадают.
#### Пинг Client2 ->  Cleint3/4 

root@ubuntu:~# ping 192.168.0.3

    PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
    ^C
    --- 192.168.0.3 ping statistics ---
    2 packets transmitted, 0 received, 100% packet loss, time 999ms

До Client 3 пинга нет, как и должно быть VNI не совпадают.

root@ubuntu:~# ping 192.168.0.4

    PING 192.168.0.4 (192.168.0.4) 56(84) bytes of data.
    64 bytes from 192.168.0.4: icmp_seq=1 ttl=64 time=216 ms
    64 bytes from 192.168.0.4: icmp_seq=2 ttl=64 time=261 ms
    64 bytes from 192.168.0.4: icmp_seq=3 ttl=64 time=308 ms
    ^C
    --- 192.168.0.4 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 216.321/262.051/308.669/37.710 ms

До Client 4 пинг есть, VNI совпадает влан нет.

### evpn database  

root@vQFX-RE-Leaf1> show evpn database   

    Instance: default-switch
    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
         100100     32:9e:e8:0d:24:0e  10.0.255.2                     Jan 17 07:47:58
         100100     36:d6:92:fe:a6:b1  xe-0/0/3.0                     Jan 17 08:48:58
         100100     50:43:a7:04:c4:00  10.0.255.2                     Jan 17 08:16:55  192.168.0.3
         100100     50:67:b9:04:c4:00  10.0.255.2                     Jan 17 07:47:58
         100100     50:92:20:04:c1:00  xe-0/0/3.0                     Jan 17 08:24:21  192.168.0.1
         300300     1e:42:e0:1c:07:bc  xe-0/0/4.0                     Jan 17 08:27:07
         300300     3e:d6:cf:35:c9:90  10.0.255.3                     Jan 18 05:44:21
         300300     50:51:05:04:c2:00  10.0.255.3                     Jan 18 05:44:21  192.168.0.4
         300300     50:93:86:04:c3:00  xe-0/0/4.0                     Jan 22 15:57:00  192.168.0.2

### Таблица маршрутизации (Оставлены только EVPN+VxLAN таблицы)

    :vxlan.inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    10.0.1.0/31        *[Direct/0] 6d 23:28:40
                        >  via xe-0/0/1.0
    10.0.1.0/32        *[Local/0] 6d 23:28:40
                          Local via xe-0/0/1.0
    10.0.2.0/31        *[Direct/0] 6d 23:28:40
                        >  via xe-0/0/2.0
    10.0.2.0/32        *[Local/0] 6d 23:28:40
                          Local via xe-0/0/2.0
    10.0.255.1/32      *[Direct/0] 6d 23:28:40
                        >  via lo0.0
    10.0.255.2/32      *[Static/1] 6d 23:06:55, metric2 0
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    10.0.255.3/32      *[Static/1] 4d 10:22:39, metric2 0
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    169.254.0.0/24     *[Direct/0] 6d 23:28:40
                        >  via em1.0
    169.254.0.2/32     *[Local/0] 6d 23:28:40
                          Local via em1.0

    bgp.evpn.0: 17 destinations, 26 routes (17 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    2:10.0.255.1:65000::100100::36:d6:92:fe:a6:b1/304 MAC/IP        
                      *[EVPN/170] 5d 07:18:02
                          Indirect
    2:10.0.255.1:65000::100100::50:92:20:04:c1:00/304 MAC/IP        
                      *[EVPN/170] 5d 07:42:47
                          Indirect
    2:10.0.255.1:65000::300300::1e:42:e0:1c:07:bc/304 MAC/IP        
                      *[EVPN/170] 5d 07:39:53
                          Indirect
    2:10.0.255.1:65000::300300::50:93:86:04:c3:00/304 MAC/IP        
                      *[EVPN/170] 5d 07:42:20
                          Indirect
    2:10.0.255.2:65000::100100::32:9e:e8:0d:24:0e/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.2:65000::100100::50:43:a7:04:c4:00/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.2:65000::100100::50:67:b9:04:c4:00/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.3:65000::300300::3e:d6:cf:35:c9:90/304 MAC/IP        
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.3:65000::300300::50:51:05:04:c2:00/304 MAC/IP        
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.1:65000::100100::50:92:20:04:c1:00::192.168.0.1/304 MAC/IP        
                      *[EVPN/170] 5d 07:42:39
                          Indirect
    2:10.0.255.1:65000::300300::50:93:86:04:c3:00::192.168.0.2/304 MAC/IP        
                      *[EVPN/170] 00:09:59
                          Indirect
    2:10.0.255.2:65000::100100::50:43:a7:04:c4:00::192.168.0.3/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.3:65000::300300::50:51:05:04:c2:00::192.168.0.4/304 MAC/IP        
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    3:10.0.255.1:65000::100100::10.0.255.1/248 IM            
                      *[EVPN/170] 6d 23:28:41
                          Indirect
    3:10.0.255.1:65000::300300::10.0.255.1/248 IM            
                      *[EVPN/170] 5d 07:58:07
                          Indirect
    3:10.0.255.2:65000::100100::10.0.255.2/248 IM            
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    3:10.0.255.3:65000::300300::10.0.255.3/248 IM            
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0

    default-switch.evpn.0: 17 destinations, 26 routes (17 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    2:10.0.255.1:65000::100100::36:d6:92:fe:a6:b1/304 MAC/IP        
                      *[EVPN/170] 5d 07:18:02
                          Indirect
    2:10.0.255.1:65000::100100::50:92:20:04:c1:00/304 MAC/IP        
                      *[EVPN/170] 5d 07:42:47
                          Indirect
    2:10.0.255.1:65000::300300::1e:42:e0:1c:07:bc/304 MAC/IP        
                      *[EVPN/170] 5d 07:39:53
                          Indirect
    2:10.0.255.1:65000::300300::50:93:86:04:c3:00/304 MAC/IP        
                      *[EVPN/170] 5d 07:42:20
                          Indirect
    2:10.0.255.2:65000::100100::32:9e:e8:0d:24:0e/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.2:65000::100100::50:43:a7:04:c4:00/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.2:65000::100100::50:67:b9:04:c4:00/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.3:65000::300300::3e:d6:cf:35:c9:90/304 MAC/IP        
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.3:65000::300300::50:51:05:04:c2:00/304 MAC/IP        
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.1:65000::100100::50:92:20:04:c1:00::192.168.0.1/304 MAC/IP        
                      *[EVPN/170] 5d 07:42:39
                          Indirect
    2:10.0.255.1:65000::300300::50:93:86:04:c3:00::192.168.0.2/304 MAC/IP        
                      *[EVPN/170] 00:09:59 
                          Indirect
    2:10.0.255.2:65000::100100::50:43:a7:04:c4:00::192.168.0.3/304 MAC/IP        
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    2:10.0.255.3:65000::300300::50:51:05:04:c2:00::192.168.0.4/304 MAC/IP        
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    3:10.0.255.1:65000::100100::10.0.255.1/248 IM            
                      *[EVPN/170] 6d 23:28:41
                          Indirect
    3:10.0.255.1:65000::300300::10.0.255.1/248 IM            
                      *[EVPN/170] 5d 07:58:07
                          Indirect
    3:10.0.255.2:65000::100100::10.0.255.2/248 IM            
                      *[BGP/170] 08:29:42, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 02:15:56, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    3:10.0.255.3:65000::300300::10.0.255.3/248 IM            
                      *[BGP/170] 17:08:20, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 04:05:07, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0

## Выполнение Cisco (Не успешно)

Вышла ошибка:
***2026 Jan 22 23:05:56 switch %$ VDC-1 %$ nve[8159]: Module 1 is not 'feature nv overlay' capable. 'feature nve overlay' requires application leaf engine' (ALE) 1 or above based line cards. Please consult documentation.***

Оформлю ДЗ, но в след раз возможно перейду на Аристу, к сожалению других образов cisco Nexus9000 у меня нет и возможности загрузить тоже.

Добавление конфигурации BGP EVPN, VxLAN
Для всех Leaf одинаковая (Кроме autonomous-system и ip neighbor, RD/RT)

+ nv overlay evpn  
feature bgp  
feature vn-segment-vlan-based  
feature bfd  
feature nv overlay  


+ router bgp 65001  
   + router-id 10.0.255.1  
  bestpath as-path multipath-relax
  reconnect-interval 12
  log-neighbor-changes
  address-family l2vpn evpn
    + maximum-paths 2  
      retain route-target all - ***Эта команда не нужна на leaf ихсходя из уроков и описанию, но иначе mac адреса не прилетают***  
  address-family ipv4 unicast  
     + redistribute direct route-map EXPORT-Loopback    
       maximum-paths 2
  + template peer OVERLAY_EVPN-VxLAN
    + remote-as 65000
     ebgp-multihop 2
     timers 3 9
     address-family l2vpn evpn  
      + send-community  
       send-community extended  
       rewrite-evpn-rt-asn
  + template peer UNDERLAY_EBGP  
    + remote-as 65000  
      timers 3 9  
      address-family ipv4 unicast  
  + neighbor 10.0.1.1
    + inherit peer UNDERLAY_EBGP
  + neighbor 10.0.2.1
    + inherit peer UNDERLAY_EBGP   
  + neighbor 10.0.1.1  
     + inherit peer SPINE  
  + neighbor 10.0.2.1  
     + inherit peer SPINE 

+ evpn  
  + vni 100100 l2  
    + rd auto  
     route-target import 65000:100100  
     route-target export 65000:100100  
  + vni 300300 l2  
    + rd auto  
     route-target import 65000:300300  
     route-target export 65000:300300  

Настройка vlan

+ vlan 100  
  + name VLAN100  
    vn-segment 100100  
+ vlan 300  
  + name VLAN300  
     vn-segment 300300  

Настройка nve 

+ interface nve1  
  + no shutdown  
    host-reachability protocol bgp  
    source-interface loopback100  
    member vni 100100  
    ingress-replication protocol bgp  
    member vni 300300  
    ingress-replication protocol bgp  


#### Политика для распространения lo 

+ route-map EXPORT-Loopback permit 10  
  match ip address prefix-list Looback_for_BGP 

+ ip prefix-list Looback_for_BGP seq 5 permit 10.0.255.0/24 eq 32 
+ ip prefix-list Looback_for_BGP seq 10 permit 10.255.255.0/24 eq 32 

Для всех Spine одинаковая (Кроме autonomous-system и ip neighbor):

#### Общие настройки  BGP EVPN


+ feature bgp
  feature vn-segment-vlan-based
  feature nv overlay
  nv overlay evpn

+ router bgp 65000    
  + router-id 10.0.0.1  
    bestpath as-path multipath-relax  
    reconnect-interval 12  
    log-neighbor-changes  
    address-family ipv4 unicast    
      + maximum-paths 2 
        redistribute direct route-map EXPORT-Loopback
  + address-family l2vpn evpn
    + maximum-paths 2
  + template peer OVERLAY_EVPN-VxLAN
    + update-source loopback0
      ebgp-multihop 2
      address-family l2vpn evpn
      + send-community
        send-community extended
        route-map NH_UNCHANGED out
        rewrite-evpn-rt-asn
  + neighbor 10.0.255.1
    + inherit peer OVERLAY_EVPN-VxLAN
      remote-as 65001
  + neighbor 10.0.255.2
    + inherit peer OVERLAY_EVPN-VxLAN
      remote-as 65002
  + neighbor 10.0.255.3
    + inherit peer OVERLAY_EVPN-VxLAN
      remote-as 65003
  + neighbor 10.0.0.0/22 remote-as route-map AS-LEAF 
    address-family ipv4 unicast  
+ route-map AS-LEAF permit 10  
   match as-number 65001-65100 

### Состояние BGP EVPN (на SPINE)

NXOS-Spine1# sh bgp l2vpn evpn summary 

    BGP summary information for VRF default, address family L2VPN EVPN
    BGP router identifier 10.0.0.1, local AS number 65000
    BGP table version is 249, L2VPN EVPN config peers 3, capable peers 3
    11 network entries and 11 paths using 2684 bytes of memory
    BGP attribute entries [8/1376], BGP AS path entries [3/18]
    BGP community entries [0/0], BGP clusterlist entries [0/0]

    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.0.255.1      4 65001   15241   15197      249    0    0 09:57:21 6         
    10.0.255.2      4 65002   14817   14812      249    0    0 09:58:29 2         
    10.0.255.3      4 65003   15236   15224      249    0    0 09:59:22 3         
    NXOS-Spine1# 

NXOS-Spine2# sh bgp l2vpn evpn summary 

    BGP summary information for VRF default, address family L2VPN EVPN
    BGP router identifier 10.0.0.2, local AS number 65000
    BGP table version is 243, L2VPN EVPN config peers 3, capable peers 3
    11 network entries and 11 paths using 2684 bytes of memory
    BGP attribute entries [8/1376], BGP AS path entries [3/18]
    BGP community entries [0/0], BGP clusterlist entries [0/0]

    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.0.255.1      4 65001   14789   14737      243    0    0 09:58:19 6         
    10.0.255.2      4 65002   14374   14368      243    0    0 10:00:20 2         
    10.0.255.3      4 65003   14812   14802      243    0    0 10:01:10 3    

Мак адреса на LEAF1 (на других аналогично):

NXOS-Leaf1# sh bgp l2vpn evpn

    BGP routing table information for VRF default, address family L2VPN EVPN
    BGP table version is 246, Local Router ID is 10.0.255.1
    Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
    Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
    njected
    Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
    est2

      Network            Next Hop            Metric     LocPrf     Weight Path
    Route Distinguisher: 10.0.255.1:32867    (L2VNI 100100)
    *>l[2]:[0]:[0]:[48]:[50ad.c904.8600]:[0]:[0.0.0.0]/216
                          10.255.255.1                      100      32768 i
    *>l[2]:[0]:[0]:[48]:[52f0.9b60.5e42]:[0]:[0.0.0.0]/216
                          10.255.255.1                      100      32768 i
    *>l[3]:[0]:[32]:[10.255.255.1]/88
                          10.255.255.1                      100      32768 i

    Route Distinguisher: 10.0.255.1:33067    (L2VNI 300300)
    *>l[2]:[0]:[0]:[48]:[3640.7e89.fc1c]:[0]:[0.0.0.0]/216
                          10.255.255.1                      100      32768 i
    *>l[2]:[0]:[0]:[48]:[504b.b204.8700]:[0]:[0.0.0.0]/216
                          10.255.255.1                      100      32768 i
    *>l[3]:[0]:[32]:[10.255.255.1]/88
                          10.255.255.1                      100      32768 i

    Route Distinguisher: 10.0.255.2:32867
    *>e[2]:[0]:[0]:[48]:[50fd.1804.8800]:[0]:[0.0.0.0]/216
                          10.255.255.2                                   0 65000 650
    02 i
    *>e[3]:[0]:[32]:[10.255.255.2]/88
                          10.255.255.2                                   0 65000 650
    02 i

    Route Distinguisher: 10.0.255.3:32967
    *>e[2]:[0]:[0]:[48]:[1ec6.cc98.bff6]:[0]:[0.0.0.0]/216
                          10.255.255.3                                   0 65000 650
    03 i
    *>e[2]:[0]:[0]:[48]:[500c.a004.8900]:[0]:[0.0.0.0]/216
                          10.255.255.3                                   0 65000 650
    03 i
    *>e[3]:[0]:[32]:[10.255.255.3]/88
                          10.255.255.3                                   0 65000 650
    03 i

Выводы по NVE интерфейсу на LEAF1 (на других аналогично)

NXOS-Leaf1# sh nve vni 

    Codes: CP - Control Plane        DP - Data Plane          
          UC - Unconfigured         SA - Suppress ARP        
          SU - Suppress Unknown Unicast 
          Xconn - Crossconnect      
          MS-IR - Multisite Ingress Replication
    
    Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
    --------- -------- ----------------- ----- ---- ------------------ -----
    nve1      100100   UnicastBGP        Up    CP   L2 [100]                
    nve1      300300   UnicastBGP        Up    CP   L2 [300]                

NXOS-Leaf1# sh interface  nve 1

    nve1 is up
    admin state is up,  Hardware: NVE
      MTU 9216 bytes
      Encapsulation VXLAN
      Auto-mdix is turned off
      RX
        ucast: 0 pkts, 0 bytes - mcast: 0 pkts, 0 bytes
      TX
        ucast: 0 pkts, 0 bytes - mcast: 0 pkts, 0 bytes


NXOS-Leaf1# sh nve peers --- ***Туннелей нет***

Пинги для Lo100 сделанных для туннелей проходят.

NXOS-Leaf1# ping 10.255.255.2

    PING 10.255.255.2 (10.255.255.2): 56 data bytes
    64 bytes from 10.255.255.2: icmp_seq=0 ttl=253 time=17.331 ms
    64 bytes from 10.255.255.2: icmp_seq=1 ttl=253 time=5.513 ms
    64 bytes from 10.255.255.2: icmp_seq=2 ttl=253 time=4.598 ms
    64 bytes from 10.255.255.2: icmp_seq=3 ttl=253 time=6.1 ms
    64 bytes from 10.255.255.2: icmp_seq=4 ttl=253 time=9.579 ms

    --- 10.255.255.2 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 4.598/8.624/17.331 ms

NXOS-Leaf1# ping 10.255.255.3

    PING 10.255.255.3 (10.255.255.3): 56 data bytes
    64 bytes from 10.255.255.3: icmp_seq=0 ttl=253 time=24.319 ms
    64 bytes from 10.255.255.3: icmp_seq=1 ttl=253 time=13.326 ms
    64 bytes from 10.255.255.3: icmp_seq=2 ttl=253 time=3.936 ms
    64 bytes from 10.255.255.3: icmp_seq=3 ttl=253 time=3.434 ms
    64 bytes from 10.255.255.3: icmp_seq=4 ttl=253 time=8.419 ms

    --- 10.255.255.3 ping statistics ---
    5 packets transmitted, 5 packets received, 0.00% packet loss
    round-trip min/avg/max = 3.434/10.686/24.319 ms