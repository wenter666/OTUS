# Д/з №6
## VxLAN. EVPN L3 // ДЗ 
Цель:
Настроить маршрутизацию в рамках Overlay между клиентами.

## Сводная таблица (Добавлена адресация Client1-4)
|  Оборудование  |      Lo0       |
| -------------- |----------------| 
|    Leaf1       | 10.0.255.1/32  | 
|    Leaf2       | 10.0.255.2/32  | 
|    Leaf3       | 10.0.255.3/32  | 
|    Spine1      | 10.0.0.1/32    |
|    Spine2      | 10.0.0.2/32    |
|    Client1     | 192.168.0.1/32 | 
|    Client2     | 192.168.1.1/32 | 
|    Client3     | 192.168.2.1/32 |


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

#### Общие настройки VRF, VxLAN, VirtualGW на LEAF

    set routing-instances Intermediate_VNI vtep-source-interface lo0.0
    set routing-instances Intermediate_VNI instance-type vrf
    set routing-instances Intermediate_VNI interface irb.200
    set routing-instances Intermediate_VNI route-distinguisher 10.0.255.2:65002 - ***10.0.255.3:65003 - LEAF2 10.0.255.3:65003 - LEAF2***
    set routing-instances Intermediate_VNI vrf-target target:1:1
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes advertise direct-nexthop
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes encapsulation vxlan
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes vni 111111

#### LEAF1

    set interfaces irb unit 100 virtual-gateway-accept-data
    set interfaces irb unit 100 family inet address 192.168.0.254/24
    set interfaces irb unit 100 family inet address 192.168.0.100/24 virtual-gateway-address 192.168.0.254
    set interfaces irb unit 100 mac 00:00:00:00:00:01

    set switch-options vtep-source-interface lo0.0
    set switch-options route-distinguisher 10.0.255.1:65000
    set switch-options vrf-target target:65000:1

    set vlans vlan100 vlan-id 100
    set vlans vlan100 l3-interface irb.100  
    set vlans vlan100 vxlan vni 100100

#### LEAF2

    set interfaces irb unit 200 virtual-gateway-accept-data
    set interfaces irb unit 200 family inet address 192.168.1.254/24
    set interfaces irb unit 200 family inet address 192.168.1.100/24 virtual-gateway-address 192.168.1.254
    set interfaces irb unit 200 mac 00:00:00:00:00:02

    set switch-options vtep-source-interface lo0.0
    set switch-options route-distinguisher 10.0.255.2:65000
    set switch-options vrf-target target:65000:1
            
    set vlans vlan200 vlan-id 200
    set vlans vlan200 l3-interface irb.200
    set vlans vlan200 vxlan vni 200200

#### LEAF3

    set protocols igmp-snooping vlan default
    set switch-options vtep-source-interface lo0.0
    set switch-options route-distinguisher 10.0.255.3:65000
    set switch-options vrf-target target:65000:1

    set vlans vlan300 vlan-id 300
    set vlans vlan300 l3-interface irb.300  
    set vlans vlan300 vxlan vni 300300

    set interfaces em1 unit 0 family inet address 169.254.0.2/24
    set interfaces irb unit 300 virtual-gateway-accept-data
    set interfaces irb unit 300 family inet address 192.168.2.254/24
    set interfaces irb unit 300 family inet address 192.168.2.100/24 virtual-gateway-address 192.168.2.254
    set interfaces irb unit 300 mac 00:00:00:00:00:03

**Конфигурация SPINE без изменений.**
  

### Состояние BGP (Только на SPINE)

root@vQFX-RE-**Spine1**> show bgp summary

    Threading mode: BGP I/O
    Groups: 2 Peers: 6 Down peers: 0
    Unconfigured peers: 3
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                          8          6          0          0          0          0
    bgp.evpn.0           
                          28         28          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.1.0              65001        177        172       0      80     1:15:56 Establ
      inet.0: 2/3/3/0
    10.0.1.2              65002       1408       1402       0      76    10:34:15 Establ
      inet.0: 2/3/3/0
    10.0.1.4              65003        527        521       0      83     3:54:13 Establ
      inet.0: 2/2/2/0
    10.0.255.1            65000      43767      43746       0       0 1w6d 19:33:38 Establ
      bgp.evpn.0: 9/9/9/0
    10.0.255.2            65000      43757      43837       0       0 1w6d 19:32:11 Establ
      bgp.evpn.0: 11/11/11/0
    10.0.255.3            65000      43752      43821       0       0 1w6d 19:27:17 Establ  
          bgp.evpn.0: 4/4/4/0

root@vQFX-RE-****Spine2****> show bgp summary     

    Threading mode: BGP I/O
    Groups: 2 Peers: 6 Down peers: 0
    Unconfigured peers: 3
    Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
    inet.0               
                          10          6          0          0          0          0
    bgp.evpn.0           
                          28         28          0          0          0          0
    Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
    10.0.2.0              65001       1329       1323       0      71     9:58:45 Establ
      inet.0: 2/3/3/0
    10.0.2.2              65002       1306       1303       0      71     9:49:37 Establ
      inet.0: 2/3/3/0
    10.0.2.4              65003         71         70       0      85       29:12 Establ
      inet.0: 2/4/4/0
    10.0.255.1            65000      43782      43756       0       0 1w6d 19:40:37 Establ
      bgp.evpn.0: 9/9/9/0
    10.0.255.2            65000      43758      43859       0       0 1w6d 19:32:46 Establ
      bgp.evpn.0: 11/11/11/0
    10.0.255.3            65000      43753      43857       0       0 1w6d 19:27:52 Establ
      bgp.evpn.0: 8/8/8/0


### Вывод evpn database

root@vQFX-RE-**Leaf1**> show evpn database   

    Instance: default-switch
    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
        100100     00:00:00:00:00:01  irb.100                        Jan 29 11:40:38  192.168.0.100
        100100     00:00:5e:00:01:01  05:00:00:fd:e8:00:01:87:04:00  Jan 29 11:40:37  192.168.0.254
        100100     36:d6:92:fe:a6:b1  xe-0/0/3.0                     Jan 17 08:48:58
        100100     50:66:1b:04:c1:00  xe-0/0/3.0                     Jan 29 11:47:43  192.168.0.1
        100100     50:92:20:04:c1:00  xe-0/0/3.0                     Jan 17 08:24:21
        200200     00:00:5e:00:01:01  05:00:00:fd:e8:00:03:0e:08:00  Jan 29 11:39:48  192.168.1.254
        200200     22:54:0d:5c:5b:af  10.0.255.2                     Jan 29 12:27:05
        200200     32:9e:e8:0d:24:0e  10.0.255.2                     Jan 29 11:24:14
        200200     50:60:b3:04:c3:00  10.0.255.2                     Jan 29 12:11:12  192.168.1.1
        200200     50:93:86:04:c3:00  10.0.255.2                     Jan 29 11:24:14  192.168.0.2
        300300     00:00:5e:00:01:01  05:00:00:fd:e8:00:04:95:0c:00  Jan 29 12:13:15  192.168.2.254
        300300     22:9c:03:77:32:f8  10.0.255.3                     Jan 29 12:20:32
        300300     50:47:e1:04:c4:00  10.0.255.3                     Jan 29 12:31:54  192.168.2.1

root@vQFX-RE-**Leaf2**> show evpn database

    Instance: default-switch
    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
        100100     00:00:5e:00:01:01  05:00:00:fd:e8:00:01:87:04:00  Jan 29 11:40:38  192.168.0.254
        100100     36:d6:92:fe:a6:b1  10.0.255.1                     Jan 29 11:28:23
        100100     50:66:1b:04:c1:00  10.0.255.1                     Jan 29 11:47:44  192.168.0.1
        100100     50:92:20:04:c1:00  10.0.255.1                     Jan 29 11:28:23
        200200     00:00:00:00:00:02  irb.200                        Jan 29 12:11:10  192.168.1.100
        200200     00:00:5e:00:01:01  05:00:00:fd:e8:00:03:0e:08:00  Jan 29 12:11:10  192.168.1.254
        200200     22:54:0d:5c:5b:af  xe-0/0/3.0                     Jan 29 12:27:05
        200200     32:9e:e8:0d:24:0e  xe-0/0/3.0                     Jan 29 11:21:32
        200200     50:60:b3:04:c3:00  xe-0/0/3.0                     Jan 29 12:11:11  192.168.1.1
        200200     50:93:86:04:c3:00  xe-0/0/3.0                     Jan 29 11:03:22  192.168.0.2
        300300     00:00:5e:00:01:01  05:00:00:fd:e8:00:04:95:0c:00  Jan 29 12:13:15  192.168.2.254
        300300     22:9c:03:77:32:f8  10.0.255.3                     Jan 29 12:20:32
        300300     50:47:e1:04:c4:00  10.0.255.3                     Jan 29 12:31:54  192.168.2.1

root@vQFX-RE-**Leaf3**> show evpn database    

    Instance: default-switch
    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
        100100     00:00:5e:00:01:01  05:00:00:fd:e8:00:01:87:04:00  Jan 29 11:40:39  192.168.0.254
        100100     36:d6:92:fe:a6:b1  10.0.255.1                     Jan 29 11:28:24
        100100     50:66:1b:04:c1:00  10.0.255.1                     Jan 29 11:47:44  192.168.0.1
        100100     50:92:20:04:c1:00  10.0.255.1                     Jan 29 11:28:24
        200200     00:00:5e:00:01:01  05:00:00:fd:e8:00:03:0e:08:00  Jan 29 11:39:48  192.168.1.254
        200200     22:54:0d:5c:5b:af  10.0.255.2                     Jan 29 12:27:06
        200200     32:9e:e8:0d:24:0e  10.0.255.2                     Jan 29 11:24:14
        200200     50:60:b3:04:c3:00  10.0.255.2                     Jan 29 12:11:13  192.168.1.1
        200200     50:93:86:04:c3:00  10.0.255.2                     Jan 29 11:24:14  192.168.0.2
        300300     00:00:00:00:00:03  irb.300                        Jan 29 12:13:15  192.168.2.100
        300300     00:00:5e:00:01:01  05:00:00:fd:e8:00:04:95:0c:00  Jan 29 12:13:15  192.168.2.254
        300300     22:9c:03:77:32:f8  xe-0/0/3.0                     Jan 29 12:20:32
        300300     50:47:e1:04:c4:00  xe-0/0/3.0                     Jan 29 12:31:54  192.168.2.1

#### Пинг Client1 ->  Cleint2/3 

root@cli:~# ping 192.168.1.1

    PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
    64 bytes from 192.168.1.1: icmp_seq=1 ttl=62 time=485 ms
    64 bytes from 192.168.1.1: icmp_seq=2 ttl=62 time=206 ms
    64 bytes from 192.168.1.1: icmp_seq=3 ttl=62 time=211 ms
    ^C
    --- 192.168.1.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms

root@cli:~# ping 192.168.2.1

    PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
    64 bytes from 192.168.2.1: icmp_seq=1 ttl=62 time=380 ms
    64 bytes from 192.168.2.1: icmp_seq=2 ttl=62 time=396 ms
    64 bytes from 192.168.2.1: icmp_seq=3 ttl=62 time=314 ms
    ^C
    --- 192.168.2.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 314.910/364.041/396.808/35.383 ms


### Таблица маршрутизации (Только VRF)

    root@vQFX-RE-Leaf1> show route table Intermediate_VNI.      

    Intermediate_VNI.inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    192.168.0.0/24     *[Direct/0] 00:57:17
                        >  via irb.100
    192.168.0.1/32     *[EVPN/7] 00:50:11
                        >  via irb.100
    192.168.0.100/32   *[Local/0] 00:57:17
                          Local via irb.100
    192.168.0.254/32   *[Local/0] 00:57:17
                          Local via irb.100
    192.168.1.0/24     *[EVPN/170] 01:09:32
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0
    192.168.2.0/24     *[EVPN/170] 00:24:39
                          to 10.0.1.1 via xe-0/0/1.0
                        >  to 10.0.2.1 via xe-0/0/2.0

    Intermediate_VNI.inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both
                                            
    ff02::2/128        *[INET6/0] 01:42:43
                          MultiRecv

    Intermediate_VNI.evpn.0: 3 destinations, 5 routes (3 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    5:10.0.255.1:65001::0::192.168.0.0::24/248               
                      *[EVPN/170] 00:57:17
                          Indirect
    5:10.0.255.2:65002::0::192.168.1.0::24/248               
                      *[BGP/170] 01:09:32, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 01:09:32, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
    5:10.0.255.3:65003::0::192.168.2.0::24/248               
                      *[BGP/170] 00:24:39, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 00:24:40, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0

root@vQFX-RE-Leaf1> show route table bgp.evpn.0 | match 192.168. 

    2:10.0.255.1:65000::100100::00:00:5e:00:01:01::192.168.0.254/304 MAC/IP        
    2:10.0.255.1:65000::100100::50:66:1b:04:c1:00::192.168.0.1/304 MAC/IP        
    2:10.0.255.2:65000::200200::00:00:5e:00:01:01::192.168.1.254/304 MAC/IP        
    2:10.0.255.2:65000::200200::50:60:b3:04:c3:00::192.168.1.1/304 MAC/IP        
    2:10.0.255.2:65000::200200::50:93:86:04:c3:00::192.168.0.2/304 MAC/IP        
    2:10.0.255.3:65000::300300::00:00:5e:00:01:01::192.168.2.254/304 MAC/IP        
    2:10.0.255.3:65000::300300::50:47:e1:04:c4:00::192.168.2.1/304 MAC/IP        
    5:10.0.255.1:65001::0::192.168.0.0::24/248               
    5:10.0.255.2:65002::0::192.168.1.0::24/248               
    5:10.0.255.3:65003::0::192.168.2.0::24/248      

## Выполнение ARISTA

Так как ДЗ на Аристах выполняется впервые, выложу полную конфигурацию оборудования.

Конфигурация LEAF

**vEOS-Leaf1#sh running-config**

    hostname vEOS-Leaf1
    !
    vlan 100
      name vlan100
    !
    vrf instance Intermediate_VNI
    !
    interface Ethernet1
      description _To_SPINE1_
      no switchport
      ip address 10.0.1.0/31
    !
    interface Ethernet2
      description _To_SPINE2_
      no switchport
      ip address 10.0.2.0/31
    !
    interface Ethernet3
      switchport access vlan 100
    !
    interface Loopback0
      ip address 10.0.255.1/32
    !
    interface Loopback1
      ip address 10.255.255.1/32
    !
    interface Management1
    !
    interface Vlan100
      vrf Intermediate_VNI
      ip address virtual 192.168.0.254/24
    !
    interface Vxlan1
      vxlan source-interface Loopback1
      vxlan udp-port 4789
      vxlan vlan 100 vni 100100
      vxlan vrf Intermediate_VNI vni 100001
      vxlan learn-restrict any
    !
    ip virtual-router mac-address 00:11:22:33:44:55
    !
    ip routing
    ip routing vrf Intermediate_VNI
    !
    router bgp 65001
      router-id 10.0.255.1
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN remote-as 65000
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor underlay peer group
      neighbor underlay remote-as 65000
      neighbor 10.0.0.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.0.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.1.1 peer group underlay
      neighbor 10.0.2.1 peer group underlay
      !
      vlan 100
          rd 65001:100100
          route-target both 100:100100
          redistribute learned
      !
      vlan 200
          rd 65001:200200
          route-target both 200:200200
          redistribute learned
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          neighbor underlay activate
          network 10.0.255.1/32
          network 10.255.255.1/32
      !
      vrf Intermediate_VNI
          rd 10.0.255.1:1
          route-target import evpn 1:1
          route-target export evpn 1:1
          redistribute connected

**vEOS-Leaf2#sh running-config**

    hostname vEOS-Leaf2
    !
    vlan 200
      name vlan200
    !
    vrf instance Intermediate_VNI
    !
    interface Ethernet1
      description _To_SPINE1_
      no switchport
      ip address 10.0.1.2/31
    !
    interface Ethernet2
      description _To_SPINE2_
      no switchport
      ip address 10.0.2.2/31
    !
    interface Ethernet3
      switchport access vlan 200
    !
    interface Loopback0
      ip address 10.0.255.2/32
    !
    interface Loopback1
      ip address 10.255.255.2/32
    !
    interface Management1
    !
    interface Vlan200
      vrf Intermediate_VNI
      ip address virtual 192.168.1.254/24
    !
    interface Vxlan1
      vxlan source-interface Loopback1
      vxlan udp-port 4789
      vxlan vlan 200 vni 200200
      vxlan vrf Intermediate_VNI vni 100001
      vxlan learn-restrict any
    !
    ip virtual-router mac-address 00:11:22:33:44:55
    !
    ip routing
    ip routing vrf Intermediate_VNI
    !
    router bgp 65002
      router-id 10.0.255.2
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN remote-as 65000
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor underlay peer group
      neighbor underlay remote-as 65000
      neighbor 10.0.0.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.0.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.1.3 peer group underlay
      neighbor 10.0.2.3 peer group underlay
      !
      vlan 200
          rd 65002:200200
          route-target both 200:200200
          redistribute learned
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          neighbor underlay activate
          network 10.0.255.2/32
          network 10.255.255.2/32
      !
      vrf Intermediate_VNI
          rd 10.0.255.2:1
          route-target import evpn 1:1
          route-target export evpn 1:1
          redistribute connected

**vEOS-Leaf3#sh running-config**

    hostname vEOS-Leaf3
    !
    vlan 300
      name vlan300
    !
    vrf instance Intermediate_VNI
    !
    interface Ethernet1
      description _To_SPINE1_
      no switchport
      ip address 10.0.1.4/31
    !
    interface Ethernet2
      description _To_SPINE2_
      no switchport
      ip address 10.0.2.4/31
    !
    interface Ethernet3
      switchport access vlan 300
    !
    interface Loopback0
      ip address 10.0.255.3/32
    !
    interface Loopback1
      ip address 10.255.255.3/32
    !
    interface Management1
    !
    interface Vlan300
      vrf Intermediate_VNI
      ip address virtual 192.168.2.254/24
    !
    interface Vxlan1
      vxlan source-interface Loopback1
      vxlan udp-port 4789
      vxlan vlan 300 vni 300300
      vxlan vrf Intermediate_VNI vni 100001
      vxlan learn-restrict any
    !
    ip virtual-router mac-address 00:11:22:33:44:55
    !
    ip routing
    ip routing vrf Intermediate_VNI
    !
    router bgp 65003
      router-id 10.0.255.3
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN remote-as 65000
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor underlay peer group
      neighbor underlay remote-as 65000
      neighbor 10.0.0.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.0.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.1.5 peer group underlay
      neighbor 10.0.2.5 peer group underlay
      !
      vlan 300
          rd 65003:300300
          route-target both 300:300300
          redistribute learned
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          neighbor underlay activate
          network 10.0.255.3/32
          network 10.255.255.3/32
      !
      vrf Intermediate_VNI
          rd 10.0.255.3:1
          route-target import evpn 1:1
          route-target export evpn 1:1
          redistribute connected

Конфигурация SPINE

**vEOS-Spine1#sh running-config**

    hostname vEOS-Spine1

    !
    interface Ethernet1
      description _To_Leaf1_
      no switchport
      ip address 10.0.1.1/31
    !
    interface Ethernet2
      description _To_Leaf2_
      no switchport
      ip address 10.0.1.3/31
    !
    interface Ethernet3
      description _To_Leaf3_
      no switchport
      ip address 10.0.1.5/31
    !
    interface Loopback0
      ip address 10.0.0.1/32
    !
    interface Management1
    !
    ip routing
    !
    router bgp 65000
      router-id 10.0.0.1
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN next-hop-unchanged
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor 10.0.1.0 remote-as 65001
      neighbor 10.0.1.2 remote-as 65002
      neighbor 10.0.1.4 remote-as 65003
      neighbor 10.0.255.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.255.1 remote-as 65001
      neighbor 10.0.255.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.255.2 remote-as 65002
      neighbor 10.0.255.3 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.255.3 remote-as 65003
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          neighbor 10.0.1.0 activate
          neighbor 10.0.1.2 activate
          neighbor 10.0.1.4 activate
          network 10.0.0.1/32

**vEOS-Spine2(config)#sh running-config**

    hostname vEOS-Spine2
    !
    interface Ethernet1
      description _To_Leaf1_
      no switchport
      ip address 10.0.2.1/31
    !
    interface Ethernet2
      description _To_Leaf2_
      no switchport
      ip address 10.0.2.3/31
    !
    interface Ethernet3
      description _To_Leaf3_
      no switchport
      ip address 10.0.2.5/31
    !
    interface Loopback0
      ip address 10.0.0.2/32
    !
    interface Management1
    !
    ip routing
    !
    router bgp 65000
      router-id 10.0.0.2
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN next-hop-unchanged
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor 10.0.2.0 remote-as 65001
      neighbor 10.0.2.2 remote-as 65002
      neighbor 10.0.2.4 remote-as 65003
      neighbor 10.0.255.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.255.1 remote-as 65001
      neighbor 10.0.255.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.255.2 remote-as 65002
      neighbor 10.0.255.3 peer group OVERLAY_EVPN-VxLAN
      neighbor 10.0.255.3 remote-as 65003
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          neighbor 10.0.2.0 activate
          neighbor 10.0.2.2 activate
          neighbor 10.0.2.4 activate
          network 10.0.0.2/32 


### Состояние BGP 

vEOS-Spine1#sh bgp summary

    BGP summary information for VRF default
    Router identifier 10.0.0.1, local AS number 65000
    Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
    ---------- ----------- ------------- ----------------------- -------------- ---------- ----------
    10.0.1.0         65001 Established   IPv4 Unicast            Negotiated              2          2
    10.0.1.2         65002 Established   IPv4 Unicast            Negotiated              2          2
    10.0.1.4         65003 Established   IPv4 Unicast            Negotiated              2          2
    10.0.255.1       65001 Established   L2VPN EVPN              Negotiated              5          5
    10.0.255.2       65002 Established   L2VPN EVPN              Negotiated              4          4
    10.0.255.3       65003 Established   L2VPN EVPN              Negotiated              5          5

vEOS-Spine2#sh bgp summary 

    BGP summary information for VRF default
    Router identifier 10.0.0.2, local AS number 65000
    Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
    ---------- ----------- ------------- ----------------------- -------------- ---------- ----------
    10.0.2.0         65001 Established   IPv4 Unicast            Negotiated              2          2
    10.0.2.2         65002 Established   IPv4 Unicast            Negotiated              2          2
    10.0.2.4         65003 Established   IPv4 Unicast            Negotiated              2          2
    10.0.255.1       65001 Established   L2VPN EVPN              Negotiated              5          5
    10.0.255.2       65002 Established   L2VPN EVPN              Negotiated              4          4
    10.0.255.3       65003 Established   L2VPN EVPN              Negotiated              5          5 

Мак адреса на LEAF1 (на других аналогично):

vEOS-Leaf1#sh bgp  evpn 

    BGP routing table information for VRF default
    Router identifier 10.0.255.1, local AS number 65001
    Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                        c - Contributing to ECMP, % - Pending best path selection
    Origin codes: i - IGP, e - EGP, ? - incomplete
    AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

              Network                Next Hop              Metric  LocPref Weight  Path
    * >      RD: 65001:100100 mac-ip 502a.c504.8600
                                    -                     -       -       0       i
    * >      RD: 65001:100100 mac-ip 502a.c504.8600 192.168.0.1
                                    -                     -       -       0       i
    * >Ec    RD: 65002:200200 mac-ip 504b.b204.8700
                                    10.255.255.2          -       100     0       65000 65002 i
    *  ec    RD: 65002:200200 mac-ip 504b.b204.8700
                                    10.255.255.2          -       100     0       65000 65002 i
    * >Ec    RD: 65002:200200 mac-ip 504b.b204.8700 192.168.1.1
                                    10.255.255.2          -       100     0       65000 65002 i
    *  ec    RD: 65002:200200 mac-ip 504b.b204.8700 192.168.1.1
                                    10.255.255.2          -       100     0       65000 65002 i
    * >Ec    RD: 65003:300300 mac-ip 50fd.1804.8800
                                    10.255.255.3          -       100     0       65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 50fd.1804.8800
                                    10.255.255.3          -       100     0       65000 65003 i
    * >Ec    RD: 65003:300300 mac-ip 50fd.1804.8800 192.168.2.1
                                    10.255.255.3          -       100     0       65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 50fd.1804.8800 192.168.2.1
                                    10.255.255.3          -       100     0       65000 65003 i
    * >      RD: 65001:100100 imet 10.255.255.1
                                    -                     -       -       0       i
    * >Ec    RD: 65002:200200 imet 10.255.255.2
                                    10.255.255.2          -       100     0       65000 65002 i
    *  ec    RD: 65002:200200 imet 10.255.255.2
                                    10.255.255.2          -       100     0       65000 65002 i
    * >Ec    RD: 65003:300300 imet 10.255.255.3
                                    10.255.255.3          -       100     0       65000 65003 i
    *  ec    RD: 65003:300300 imet 10.255.255.3
                                    10.255.255.3          -       100     0       65000 65003 i
    * >      RD: 10.0.255.1:1 ip-prefix 192.168.0.0/24
                                    -                     -       -       0       i
    * >Ec    RD: 10.0.255.2:1 ip-prefix 192.168.1.0/24
                                    10.255.255.2          -       100     0       65000 65002 i
    *  ec    RD: 10.0.255.2:1 ip-prefix 192.168.1.0/24
                                    10.255.255.2          -       100     0       65000 65002 i
    * >Ec    RD: 10.0.255.3:1 ip-prefix 192.168.2.0/24
                                    10.255.255.3          -       100     0       65000 65003 i
    *  ec    RD: 10.0.255.3:1 ip-prefix 192.168.2.0/24
                                    10.255.255.3          -       100     0       65000 65003 i

Пример маршрута от LEAF3 к LEAF1

    BGP routing table entry for mac-ip 50fd.1804.8800 192.168.2.1, Route Distinguisher: 65003:300300
    Paths: 2 available
      65000 65003
        10.255.255.3 from 10.0.0.2 (10.0.0.2)
          Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP head, ECMP, best, ECMP contributor
          Extended Community: ***Route-Target-AS:1:1 Route-Target-AS:300:300300*** TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:9e:96:3f:31:64
          VNI: 300300 L3 VNI: 100001 ESI: 0000:0000:0000:0000:0000
      65000 65003
        10.255.255.3 from 10.0.0.1 (10.0.0.1)
          Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP, ECMP contributor
          Extended Community: ***Route-Target-AS:1:1 Route-Target-AS:300:300300*** TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:9e:96:3f:31:64
          VNI: 300300 L3 VNI: 100001 ESI: 0000:0000:0000:0000:0000

Вывод по vxlan1 интерфейсу на LEAF1

vEOS-Leaf1# sh interfaces vxlan1 

    Vxlan1 is up, line protocol is up (connected)
      Hardware is Vxlan
      Source interface is Loopback1 and is active with 10.255.255.1
      Listening on UDP port 4789
      Replication/Flood Mode is headend with Flood List Source: EVPN
      Remote MAC learning via EVPN
      VNI mapping to VLANs
      Static VLAN to VNI mapping is 
        [100, 100100]    
      Dynamic VLAN to VNI mapping for 'evpn' is
        [4097, 100001]   
      Note: All Dynamic VLANs used by VCS are internal VLANs.
            Use 'show vxlan vni' for details.
      Static VRF to VNI mapping is 
      [Intermediate_VNI, 100001]
      Shared Router MAC is 0000.0000.0000


#### Пинг Client1 ->  Cleint2/3 

root@ubuntu:~# ping 192.168.1.1

    PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
    64 bytes from 192.168.1.1: icmp_seq=1 ttl=62 time=12.9 ms
    64 bytes from 192.168.1.1: icmp_seq=2 ttl=62 time=9.75 ms
    64 bytes from 192.168.1.1: icmp_seq=3 ttl=62 time=7.89 ms
    ^C
    --- 192.168.1.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 7.899/10.216/12.996/2.109 ms

root@ubuntu:~# ping 192.168.2.1

    PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
    64 bytes from 192.168.2.1: icmp_seq=1 ttl=62 time=9.33 ms
    64 bytes from 192.168.2.1: icmp_seq=2 ttl=62 time=22.5 ms
    64 bytes from 192.168.2.1: icmp_seq=3 ttl=62 time=10.2 ms
    ^C
    --- 192.168.2.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 9.330/14.026/22.529/6.023 ms