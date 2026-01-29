# Д/з №7
## VXLAN. Multihoming
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
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab6/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0.png)

## Сводная таблица AS
|  Оборудование  |      Lo0      |   AS  |
| -------------- |---------------|-------|
|    Leaf1       | 10.0.255.1/32 | 65001 |
|    Leaf2       | 10.0.255.2/32 | 65002 |
|    Leaf3       | 10.0.255.3/32 | 65003 |
|    Spine1      | 10.0.0.1/32   | 65000 |
|    Spine2      | 10.0.0.2/32   | 65000 |


## Выполнение Juniper

#### Общие настройки добавленные на Leaf1 и leaf2

#### LEAF1

    set interfaces xe-0/0/3 description ae-multihomig
    set interfaces xe-0/0/3 gigether-options 802.3ad ae1

    set interfaces ae1 esi 00:01:01:01:01:01:01:01:01:01
    set interfaces ae1 esi all-active
    set interfaces ae1 aggregated-ether-options lacp active
    set interfaces ae1 aggregated-ether-options lacp system-id 01:01:01:01:01:01
    set interfaces ae1 unit 0 family ethernet-switching interface-mode trunk
    set interfaces ae1 unit 0 family ethernet-switching vlan members all
    set interfaces ae1 unit 0 family ethernet-switching vlan members vlan100

    set interfaces irb unit 100 virtual-gateway-accept-data
    set interfaces irb unit 100 family inet address 192.168.0.254/24
    set interfaces irb unit 100 family inet address 192.168.0.100/24 virtual-gateway-address 192.168.0.254
    set interfaces irb unit 100 mac 00:00:00:00:00:01

    set routing-instances Intermediate_VNI interface irb.100
    set vlans vlan100 l3-interface irb.100

#### LEAF2
    set interfaces xe-0/0/3 description ae-multihomig
    set interfaces xe-0/0/3 gigether-options 802.3ad ae1
    set interfaces ae1 esi 00:01:01:01:01:01:01:01:01:01
    set interfaces ae1 esi all-active
    set interfaces ae1 aggregated-ether-options lacp active
    set interfaces ae1 aggregated-ether-options lacp system-id 01:01:01:01:01:01
    set interfaces ae1 unit 0 family ethernet-switching interface-mode trunk
    set interfaces ae1 unit 0 family ethernet-switching vlan members all
    set interfaces ae1 unit 0 family ethernet-switching vlan members vlan100

    set interfaces irb unit 100 virtual-gateway-accept-data
    set interfaces irb unit 100 family inet address 192.168.0.254/24
    set interfaces irb unit 100 family inet address 192.168.0.100/24 virtual-gateway-address 192.168.0.254
    set interfaces irb unit 100 mac 00:00:00:00:00:01

    set routing-instances Intermediate_VNI interface irb.100
    set vlans vlan100 l3-interface irb.100


**Конфигурация SPINE без изменений.**
  
### Вывод evpn database

root@vQFX-RE-**Leaf1**> show evpn database   

    Instance: default-switch
    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
        100100     00:00:00:00:00:01  05:00:00:fd:e8:00:01:87:04:00  Jan 29 16:11:59  192.168.0.254
        100100     02:05:86:71:e0:00  irb.100                        Jan 29 16:10:16  192.168.0.100
        100100     50:66:1b:04:c1:00  00:01:01:01:01:01:01:01:01:01  Jan 29 15:40:12
        100100     56:b6:28:06:bd:ce  00:01:01:01:01:01:01:01:01:01  Jan 29 15:43:41
        200200     00:00:5e:00:01:01  05:00:00:fd:e8:00:03:0e:08:00  Jan 29 15:40:02  192.168.1.254
        300300     00:00:00:00:00:03  05:00:00:fd:e8:00:04:95:0c:00  Jan 29 15:40:02  192.168.2.254
        300300     22:9c:03:77:32:f8  10.0.255.3                     Jan 29 15:40:02
        300300     50:47:e1:04:c4:00  10.0.255.3                     Jan 29 15:40:02  192.168.2.1

root@vQFX-RE-**Leaf2**> show evpn database

    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
        100100     00:00:00:00:00:01  05:00:00:fd:e8:00:01:87:04:00  Jan 29 16:11:58  192.168.0.254
        100100     02:05:86:71:6c:00  irb.100                        Jan 29 16:11:58  192.168.0.100
        100100     50:66:1b:04:c1:00  00:01:01:01:01:01:01:01:01:01  Jan 29 15:41:14
        100100     56:b6:28:06:bd:ce  00:01:01:01:01:01:01:01:01:01  Jan 29 15:43:42
        200200     00:00:00:00:00:02  irb.200                        Jan 29 14:56:14  192.168.1.100
        200200     00:00:5e:00:01:01  05:00:00:fd:e8:00:03:0e:08:00  Jan 29 14:56:14  192.168.1.254
        300300     00:00:00:00:00:03  05:00:00:fd:e8:00:04:95:0c:00  Jan 29 15:37:16  192.168.2.254
        300300     22:9c:03:77:32:f8  10.0.255.3                     Jan 29 12:20:32
        300300     50:47:e1:04:c4:00  10.0.255.3                     Jan 29 15:37:17  192.168.2.1

root@vQFX-RE-**Leaf3**> show evpn database    

    Instance: default-switch
    VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
        100100     00:00:00:00:00:01  05:00:00:fd:e8:00:01:87:04:00  Jan 29 16:11:59  192.168.0.254
        100100     50:66:1b:04:c1:00  00:01:01:01:01:01:01:01:01:01  Jan 29 15:41:15
        100100     56:b6:28:06:bd:ce  00:01:01:01:01:01:01:01:01:01  Jan 29 15:43:42
        200200     00:00:5e:00:01:01  05:00:00:fd:e8:00:03:0e:08:00  Jan 29 14:56:15  192.168.1.254
        300300     00:00:00:00:00:03  05:00:00:fd:e8:00:04:95:0c:00  Jan 29 15:37:16  192.168.2.254
        300300     02:05:86:71:bb:00  irb.300                        Jan 29 15:37:16  192.168.2.100
        300300     22:9c:03:77:32:f8  xe-0/0/3.0                     Jan 29 12:20:32
        300300     50:47:e1:04:c4:00  xe-0/0/3.0                     Jan 29 15:37:17  192.168.2.1

#### Пинг Client1 ->  Cleint3 

root@cli:~# ping 192.168.2.1

    PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
    64 bytes from 192.168.2.1: icmp_seq=1 ttl=62 time=482 ms
    64 bytes from 192.168.2.1: icmp_seq=2 ttl=62 time=502 ms
    64 bytes from 192.168.2.1: icmp_seq=3 ttl=62 time=438 ms
    64 bytes from 192.168.2.1: icmp_seq=4 ttl=62 time=531 ms
    ^C
    --- 192.168.2.1 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3001ms
    rtt min/avg/max/mdev = 438.090/488.819/531.414/34.003 ms

### Вывод LACP на промежуточном оборудовании.

root@ae_bridge> show lacp interfaces ae1 extensive 

      Aggregated interface: ae1
          LACP state:       Role   Exp   Def  Dist  Col  Syn  Aggr  Timeout  Activity
            ge-0/0/1       Actor    No    No   Yes  Yes  Yes   Yes     Fast    Active
            ge-0/0/1     Partner    No    No   Yes  Yes  Yes   Yes     Fast    Active
            ge-0/0/0       Actor    No    No   Yes  Yes  Yes   Yes     Fast    Active
            ge-0/0/0     Partner    No    No   Yes  Yes  Yes   Yes     Fast    Active
          LACP protocol:        Receive State  Transmit State          Mux State 
            ge-0/0/1                  Current   Fast periodic Collecting distributing
            ge-0/0/0                  Current   Fast periodic Collecting distributing
          LACP info:        Role     System             System       Port     Port    Port 
                                  priority         identifier   priority   number     key 
            ge-0/0/1       Actor        127  2c:6b:f5:f3:d5:c0        127        2       2
            ge-0/0/1     Partner        127  01:01:01:01:01:01        127        1       2
            ge-0/0/0       Actor        127  2c:6b:f5:f3:d5:c0        127        1       2
            ge-0/0/0     Partner        127  01:01:01:01:01:01        127        1       2


### Выводы EVPN (DF LEAF1)
root@vQFX-RE-Leaf1> show evpn instance designated-forwarder 

    Instance: default-switch
      Number of ethernet segments: 4
        ESI: 00:01:01:01:01:01:01:01:01:01
          Designated forwarder: 10.0.255.1 
        ESI: 05:00:00:fd:e8:00:01:87:04:00
        ESI: 05:00:00:fd:e8:00:03:0e:08:00
        ESI: 05:00:00:fd:e8:00:04:95:0c:00

root@vQFX-RE-Leaf1> show evpn instance esi-info    

Instance: __default_evpn__

    Instance: default-switch
      Number of ethernet segments: 4
        ESI: 00:01:01:01:01:01:01:01:01:01
          Status: Resolved by IFL ae1.0
          Local interface: ae1.0, Status: Up/Forwarding
          Number of remote PEs connected: 1
            Remote PE        MAC label  Aliasing label  Mode
            10.0.255.2       100100     0               all-active   
          DF Election Algorithm: MOD based
          Designated forwarder: 10.0.255.1
          Backup forwarder: 10.0.255.2
          Last designated forwarder update: Jan 29 15:40:12


        ESI: 05:00:00:fd:e8:00:01:87:04:00
          Local interface: irb.100, Status: Up/Forwarding
          Number of remote PEs connected: 1
            Remote PE        MAC label  Aliasing label  Mode
            10.0.255.2       100100     0               all-active   
        ESI: 05:00:00:fd:e8:00:03:0e:08:00
          Status: Resolved
          Number of remote PEs connected: 1
            Remote PE        MAC label  Aliasing label  Mode
            10.0.255.2       200200     0               all-active   
        ESI: 05:00:00:fd:e8:00:04:95:0c:00
          Status: Resolved
          Number of remote PEs connected: 1
            Remote PE        MAC label  Aliasing label  Mode
            10.0.255.3       300300     0               all-active  

### Таблица маршрутизации 

root@vQFX-RE-Leaf1> show route table bgp.evpn.0 | match ^1: 

    1:10.0.255.1:0::010101010101010101::FFFF:FFFF/192 AD/ESI        
    1:10.0.255.1:0::050000fde80001870400::FFFF:FFFF/192 AD/ESI        
    1:10.0.255.1:65000::010101010101010101::0/192 AD/EVI        
    1:10.0.255.2:0::010101010101010101::FFFF:FFFF/192 AD/ESI        
    1:10.0.255.2:0::050000fde80001870400::FFFF:FFFF/192 AD/ESI        
    1:10.0.255.2:0::050000fde800030e0800::FFFF:FFFF/192 AD/ESI        
    1:10.0.255.2:65000::010101010101010101::0/192 AD/EVI        
    1:10.0.255.3:0::050000fde80004950c00::FFFF:FFFF/192 AD/ESI      

root@vQFX-RE-Leaf1> show route table __default_evpn__.evpn.0    


    __default_evpn__.evpn.0: 4 destinations, 5 routes (4 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    1:10.0.255.1:0::010101010101010101::FFFF:FFFF/192 AD/ESI        
                      *[EVPN/170] 00:38:51
                          Indirect
    1:10.0.255.1:0::050000fde80001870400::FFFF:FFFF/192 AD/ESI        
                      *[EVPN/170] 00:08:48
                          Indirect
    4:10.0.255.1:0::010101010101010101:10.0.255.1/296 ES            
                      *[EVPN/170] 00:38:52
                          Indirect
    4:10.0.255.2:0::010101010101010101:10.0.255.2/296 ES            
                      *[BGP/170] 00:38:58, localpref 100, from 10.0.0.1
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                        [BGP/170] 00:39:02, localpref 100, from 10.0.0.2
                          AS path: I, validation-state: unverified
                        >  to 10.0.1.1 via xe-0/0/1.0
                          to 10.0.2.1 via xe-0/0/2.0
                                            

## Выполнение ARISTA

Конфигурация LEAF

**vEOS-Leaf1#**

    interface Port-Channel1
      switchport mode trunk
      !
      evpn ethernet-segment
          identifier 0000:0101:0101:0101:0101
          route-target import 01:01:01:01:01:01
      lacp system-id 0101.0101.0101

    interface Ethernet3
      channel-group 1 mode active

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

**vEOS-Leaf2#**

    interface Port-Channel1
      switchport mode trunk
      !
      evpn ethernet-segment
          identifier 0000:0101:0101:0101:0101
          route-target import 01:01:01:01:01:01
      lacp system-id 0101.0101.0101
    !
    interface Ethernet3
      channel-group 1 mode active

    interface Vxlan1
      vxlan source-interface Loopback1
      vxlan udp-port 4789
      vxlan vlan 100 vni 100100
      vxlan vrf Intermediate_VNI vni 100001
      vxlan learn-restrict any

### Выводы ромежуточного оборудоваиня LACP 

localhost#sh lacp peer

    State: A = Active, P = Passive; S=ShortTimeout, L=LongTimeout;
          G = Aggregable, I = Individual; s+=InSync, s-=OutOfSync;
          C = Collecting, X = state machine expired,
          D = Distributing, d = default neighbor state
                    |                        Partner                              
    Port    Status  | Sys-id                    Port#   State     OperKey  PortPri
    ------ ----------|------------------------- ------- --------- --------- -------
    Port Channel Port-Channel1:                                            
    Et1     Bundled | 8000,01-01-01-01-01-01        3   ALGs+CD    0x0001    32768
    Et2     Bundled | 8000,01-01-01-01-01-01        3   ALGs+CD    0x0001    32768




vEOS-Leaf1(config-if-Po1)#sh bgp evpn route-type auto-discovery 

    BGP routing table information for VRF default
    Router identifier 10.0.255.1, local AS number 65001
    Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                        c - Contributing to ECMP, % - Pending best path selection
    Origin codes: i - IGP, e - EGP, ? - incomplete
    AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

              Network                Next Hop              Metric  LocPref Weight  Path
    * >      RD: 65001:100100 auto-discovery 0 0000:0101:0101:0101:0101
                                    -                     -       -       0       i
    * >      RD: 10.255.255.1:1 auto-discovery 0000:0101:0101:0101:0101
                                    -                     -       -       0       i
    * >Ec    RD: 10.255.255.2:1 auto-discovery 0000:0101:0101:0101:0101
                                    10.255.255.2          -       100     0       65000 65002 i
    *  ec    RD: 10.255.255.2:1 auto-discovery 0000:0101:0101:0101:0101
                                    10.255.255.2          -       100     0       65000 65002 i

vEOS-Leaf1(config-if-Po1)#sh bgp evpn route-type ethernet-segment 

    BGP routing table information for VRF default
    Router identifier 10.0.255.1, local AS number 65001
    Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                        c - Contributing to ECMP, % - Pending best path selection
    Origin codes: i - IGP, e - EGP, ? - incomplete
    AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

              Network                Next Hop              Metric  LocPref Weight  Path
    * >      RD: 10.255.255.1:1 ethernet-segment 0000:0101:0101:0101:0101 10.255.255.1
                                    -                     -       -       0       i
    * >Ec    RD: 10.255.255.2:1 ethernet-segment 0000:0101:0101:0101:0101 10.255.255.2
                                    10.255.255.2          -       100     0       65000 65002 i
    *  ec    RD: 10.255.255.2:1 ethernet-segment 0000:0101:0101:0101:0101 10.255.255.2
                                    10.255.255.2          -       100     0       65000 65002 i


####  DF LEAF1

vEOS-Leaf1#sh bgp  evpn instance 

    EVPN instance: VLAN 100
      Route distinguisher: 65001:100100
      Route target import: Route-Target-AS:100:100100
      Route target export: Route-Target-AS:100:100100
      Service interface: VLAN-based
      Local VXLAN IP address: 10.255.255.1
      VXLAN: enabled
      MPLS: disabled
      Local ethernet segment:
        ESI: 0000:0101:0101:0101:0101
          Type: 0 (administratively configured)
          Interface: Port-Channel1
          Mode: all-active
          State: up
          ES-Import RT: 01:01:01:01:01:01
          DF election algorithm: modulus
          Designated forwarder: 10.255.255.1
          Non-Designated forwarder: 10.255.255.2


#### Пинг Client1 ->  Cleint3 

root@ubuntu:~# ping 192.168.2.1

    PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
    64 bytes from 192.168.2.1: icmp_seq=1 ttl=62 time=12.9 ms
    64 bytes from 192.168.2.1: icmp_seq=2 ttl=62 time=11.0 ms
    64 bytes from 192.168.2.1: icmp_seq=3 ttl=62 time=11.6 ms
    64 bytes from 192.168.2.1: icmp_seq=4 ttl=62 time=10.3 ms
    ^C
    --- 192.168.2.1 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3005ms
    rtt min/avg/max/mdev = 10.328/11.473/12.901/0.949 ms