# Д/з №8
## ДЗ: VxLAN. Routing.
Цель:
Реализовать передачу суммарных префиксов через EVPN route-type 5.

## Сводная таблица ARISTA
|  Оборудование  |      Lo0       |
| -------------- |----------------| 
|    Leaf1       | 20.0.255.1/32  | 
|    Leaf2       | 20.0.255.2/32  | 
|    Leaf3       | 20.0.255.3/32  | 
|    Spine1      | 20.0.0.1/32    |
|    Spine2      | 20.0.0.2/32    |
|    Client1     | 192.168.0.2/32 | 


|   Spine ip+1   |      p2p      |   Leaf ip+0    |
|----------------|---------------|----------------|
|     Spine1     |               |                |
|      eth1      |  20.0.1.0/31  |   eth1 leaf1   |
|      eth2      |  20.0.1.2/31  |   eth1 leaf2   |    
|      eth2      |  20.0.1.4/31  |   eth1 leaf3   |
|                |               |                |
|     Spine2     |               |                |
|      eth1      |  20.0.2.0/31  |   eth2 leaf1   |
|      eth2      |  20.0.2.2/31  |   eth2 leaf2   |    
|      eth3      |  20.0.2.4/31  |   eth2 leaf3   |

## Сводная таблица BorderLeaf + EDGE router 

|        BorderLeaf      |        p2p       |      EDGE      |
|------------------------|------------------|----------------|
|  ARISTA Leaf3 Vlan4001 |  10.10.10.10/30  |  ge-0/0/0.4001 |
|  ARISTA Leaf3 Vlan4002 |  20.20.20.20/30  |  ge-0/0/0.4002 |


## Схема
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab8/%D0%A1%D1%85%D0%B5%D0%BC%D0%B01.png)|

## Сводная таблица AS ARISTA
|  Оборудование  |      Lo0      |   AS  |
| -------------- |---------------|-------|
|    Leaf1       | 20.0.255.1/32 | 64001 |
|    Leaf2       | 20.0.255.2/32 | 64002 |
|    Leaf3       | 20.0.255.3/32 | 64003 |
|    Spine1      | 20.0.0.1/32   | 64000 |
|    Spine2      | 20.0.0.2/32   | 64000 |

## Сводная таблица AS EGDE

|  Оборудование  |         Lo0        |   AS  |
| -------------- |--------------------|-------|
|    EGDE        | 100.100.100.100/32 | 63000 |

## Выполнение Общее

### LEAF1 JUNIPER

        set interfaces ge-0/0/0 description To_ARISTA_Leaf1
        set interfaces ge-0/0/0 flexible-vlan-tagging
        set interfaces ge-0/0/0 mtu 9000
        set interfaces ge-0/0/0 encapsulation flexible-ethernet-services
        set interfaces ge-0/0/0 unit 4001 description To_TENANT1
        set interfaces ge-0/0/0 unit 4001 vlan-id 4001
        set interfaces ge-0/0/0 unit 4001 family inet address 10.10.10.2/30
        set interfaces ge-0/0/0 unit 4002 description To_TENANT2
        set interfaces ge-0/0/0 unit 4002 vlan-id 4002
        set interfaces ge-0/0/0 unit 4002 family inet address 20.20.20.2/30

        set interfaces lo0 unit 0 family inet address 100.100.100.100/32 primary
        set interfaces lo0 unit 100 family inet address 8.8.8.8/32 -- ***Для эмуляции выхода нанаржу***

        set policy-options policy-statement LEAK_FROM_TEST from instance TEST
        set policy-options policy-statement LEAK_FROM_TEST from protocol direct
        set policy-options policy-statement LEAK_FROM_TEST then accept
        set policy-options policy-statement SEND-DEFAULT term 1 from route-filter 0.0.0.0/0 exact - ***Генерация дефолта***
        set policy-options policy-statement SEND-DEFAULT term 1 then accept
        set policy-options policy-statement SEND-DEFAULT term 2 then reject

        set routing-instances TEST instance-type virtual-router
        set routing-instances TEST interface lo0.100

        set routing-options router-id 100.100.100.100
        set routing-options autonomous-system 63000

        set routing-options static route 0.0.0.0/0 discard -- ***Генерация дефолта***
        set routing-options instance-import LEAK_FROM_TEST

        set protocols bgp group TO_COD log-updown
        set protocols bgp group TO_COD export SEND-DEFAULT
        set protocols bgp group TO_COD graceful-restart
        set protocols bgp group TO_COD as-override
        set protocols bgp group TO_COD neighbor 20.20.20.1 peer-as 64003
        set protocols bgp group TO_COD neighbor 10.10.10.1 peer-as 64003

### LEAF1 ARISTA

        hostname vEOS-Leaf1
        !
        vlan 100
        name vlan100
        !
        vrf instance TENANT100
        !
        interface Ethernet1
        description _To_SPINE1_
        mtu 9000
        no switchport
        ip address 20.0.1.0/31
        !
        interface Ethernet2
        description _To_SPINE2_
        mtu 9000
        no switchport
        ip address 20.0.2.0/31
        !
        interface Ethernet3
        mtu 9000
        switchport access vlan 100
        !
        interface Loopback0
        ip address 20.0.255.1/32
        !
        interface Loopback1
        ip address 20.255.255.1/32
        !
        interface Management1
        !
        interface Vlan100
        vrf TENANT100
        ip address virtual 192.168.0.254/24
        !
        interface Vxlan1
        vxlan source-interface Loopback1
        vxlan udp-port 4789
        vxlan vlan 100 vni 100100
        vxlan vrf TENANT100 vni 111111
        vxlan learn-restrict any
        !
        ip virtual-router mac-address 00:00:00:00:00:01
        !
        ip routing
        ip routing vrf TENANT100
        !
        router bgp 64001
        router-id 20.0.255.1
        no bgp default ipv4-unicast
        distance bgp 20 200 200
        maximum-paths 4 ecmp 64
        neighbor OVERLAY_EVPN-VxLAN peer group
        neighbor OVERLAY_EVPN-VxLAN remote-as 64000
        neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
        neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
        neighbor OVERLAY_EVPN-VxLAN send-community extended
        neighbor underlay peer group
        neighbor underlay remote-as 64000
        neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
        neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
        neighbor 20.0.1.1 peer group underlay
        neighbor 20.0.2.1 peer group underlay
        !
        vlan 100
            rd 64001:100100
            route-target both 100:100100
            redistribute learned
        !
        address-family evpn
            neighbor OVERLAY_EVPN-VxLAN activate
        !
        address-family ipv4
            neighbor underlay activate
            network 20.0.255.1/32
            network 20.255.255.1/32
        !
        vrf TENANT100
            rd 20.0.255.1:100
            route-target import evpn 100:100
            route-target export evpn 100:100
            redistribute connected
    !
### LEAF2 ARISTA

        hostname vEOS-Leaf2
        !
        vlan 200
        name vlan200
        !
        vrf instance TENANT200
        !
        interface Ethernet1
        description _To_SPINE1_
        mtu 9000
        no switchport
        ip address 20.0.1.2/31
        !
        interface Ethernet2
        description _To_SPINE2_
        mtu 9000
        no switchport
        ip address 20.0.2.2/31
        !
        interface Ethernet3
        mtu 9000
        switchport access vlan 200
        !
        interface Loopback0
        ip address 20.0.255.2/32
        !
        interface Loopback1
        ip address 20.255.255.2/32
        !
        interface Management1
        !
        interface Vlan200
        vrf TENANT200
        ip address virtual 192.168.2.254/24
        !
        interface Vxlan1
        vxlan source-interface Loopback1
        vxlan udp-port 4789
        vxlan vlan 200 vni 200200
        vxlan vrf TENANT200 vni 222222
        vxlan learn-restrict any
        !
        ip virtual-router mac-address 00:00:00:00:00:02
        !
        ip routing
        ip routing vrf TENANT200
        !
        router bgp 64002
        router-id 20.0.255.2
        no bgp default ipv4-unicast
        distance bgp 20 200 200
        maximum-paths 4 ecmp 64
        neighbor OVERLAY_EVPN-VxLAN peer group
        neighbor OVERLAY_EVPN-VxLAN remote-as 64000
        neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
        neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
        neighbor OVERLAY_EVPN-VxLAN send-community extended
        neighbor underlay peer group
        neighbor underlay remote-as 64000
        neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
        neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
        neighbor 20.0.1.3 peer group underlay
        neighbor 20.0.2.3 peer group underlay
        !
        vlan 200
            rd 64002:200200
            route-target both 200:200200
            redistribute learned
        !
        address-family evpn
            neighbor OVERLAY_EVPN-VxLAN activate
        !
        address-family ipv4
            neighbor underlay activate
            network 20.0.255.2/32
            network 20.255.255.2/32
        !
        vrf TENANT200
            rd 20.0.255.2:200
            route-target import evpn 200:200
            route-target export evpn 200:200
            redistribute connected


### LEAF3 ARISTA

        hostname vEOS-Leaf3
        
        !
        vlan 100
        name vlan100
        !
        vlan 200
        name vlan200
        !
        vlan 4001
        name vlan4001
        !
        vlan 4002
        name vlan4002
        !
        vrf instance TENANT100
        !
        vrf instance TENANT200
        !
        interface Ethernet1
        description _To_SPINE1_
        mtu 9000
        no switchport
        ip address 20.0.1.4/31
        !
        interface Ethernet2
        description _To_SPINE2_
        mtu 9000
        no switchport
        ip address 20.0.2.4/31
        !
        interface Ethernet3
        mtu 9000
        switchport trunk allowed vlan 4001-4002
        switchport mode trunk
        !
        interface Loopback0
        ip address 20.0.255.3/32
        !
        interface Loopback1
        ip address 20.255.255.3/32
        !
        interface Management1
        !
        interface Vlan4001
        vrf TENANT100
        ip address 10.10.10.1/30
        !
        interface Vlan4002
        vrf TENANT200
        ip address 20.20.20.1/30
        !
        interface Vxlan1
        vxlan source-interface Loopback1
        vxlan udp-port 4789
        vxlan vrf TENANT100 vni 111111
        vxlan vrf TENANT200 vni 222222
        vxlan learn-restrict any
        !
        ip routing
        ip routing vrf TENANT100
        ip routing vrf TENANT200
        !
        router bgp 64003
        router-id 20.0.255.3
        no bgp default ipv4-unicast
        distance bgp 20 200 200
        maximum-paths 4 ecmp 64
        neighbor OVERLAY_EVPN-VxLAN peer group
        neighbor OVERLAY_EVPN-VxLAN remote-as 64000
        neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
        neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
        neighbor OVERLAY_EVPN-VxLAN send-community extended
        neighbor underlay peer group
        neighbor underlay remote-as 64000
        neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
        neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
        neighbor 20.0.1.5 peer group underlay
        neighbor 20.0.2.5 peer group underlay
        !
        address-family evpn
            neighbor OVERLAY_EVPN-VxLAN activate
        !
        address-family ipv4
            neighbor underlay activate
            network 20.0.255.3/32
            network 20.255.255.3/32
        !
        vrf TENANT100
            rd 20.0.255.3:100
            route-target import evpn 100:100
            route-target export evpn 100:100
            neighbor 10.10.10.2 remote-as 63000
            !
            address-family ipv4
                neighbor 10.10.10.2 activate
        !
        vrf TENANT200
            rd 20.0.255.3:200
            route-target import evpn 200:200
            route-target export evpn 200:200
            neighbor 20.20.20.2 remote-as 63000
            !
            address-family ipv4
                neighbor 20.20.20.2 activate
!
### Таблица маршрутизации

**vEOS-Leaf1**#sh ip route vrf TENANT100

        Gateway of last resort:
        B E      0.0.0.0/0 [20/0]
                via VTEP 20.255.255.3 VNI 111111 router-mac 50:14:2f:6e:f9:da local-interface Vxlan1

        C        192.168.0.0/24
                directly connected, Vlan100

**vEOS-Leaf1**#sh bgp evpn route-type ip-prefix  ipv4

        BGP routing table information for VRF default
        Router identifier 20.0.255.1, local AS number 64001
        Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                            c - Contributing to ECMP, % - Pending best path selection
        Origin codes: i - IGP, e - EGP, ? - incomplete
        AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

                Network                Next Hop              Metric  LocPref Weight  Path
        * >Ec    RD: 20.0.255.3:100 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        *  ec    RD: 20.0.255.3:100 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        * >Ec    RD: 20.0.255.3:200 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        *  ec    RD: 20.0.255.3:200 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        * >      RD: 20.0.255.1:100 ip-prefix 192.168.0.0/24
                                        -                     -       -       0       i
        * >Ec    RD: 20.0.255.2:200 ip-prefix 192.168.2.0/24
                                        20.255.255.2          -       100     0       64000 64002 i
        *  ec    RD: 20.0.255.2:200 ip-prefix 192.168.2.0/24
                                        20.255.255.2          -       100     0       64000 64002 i


**vEOS-Leaf2**#sh ip route  vrf  TENANT200

        Gateway of last resort:
        B E      0.0.0.0/0 [20/0]
                via VTEP 20.255.255.3 VNI 222222 router-mac 50:14:2f:6e:f9:da local-interface Vxlan1

        C        192.168.2.0/24
                directly connected, Vlan200


**vEOS-Leaf2**#sh bgp evpn route-type ip-prefix  ipv4

        BGP routing table information for VRF default
        Router identifier 20.0.255.2, local AS number 64002
        Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                            c - Contributing to ECMP, % - Pending best path selection
        Origin codes: i - IGP, e - EGP, ? - incomplete
        AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

                Network                Next Hop              Metric  LocPref Weight  Path
        * >Ec    RD: 20.0.255.3:100 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        *  ec    RD: 20.0.255.3:100 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        * >Ec    RD: 20.0.255.3:200 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        *  ec    RD: 20.0.255.3:200 ip-prefix 0.0.0.0/0
                                        20.255.255.3          -       100     0       64000 64003 63000 i
        * >Ec    RD: 20.0.255.1:100 ip-prefix 192.168.0.0/24
                                        20.255.255.1          -       100     0       64000 64001 i
        *  ec    RD: 20.0.255.1:100 ip-prefix 192.168.0.0/24
                                        20.255.255.1          -       100     0       64000 64001 i
        * >      RD: 20.0.255.2:200 ip-prefix 192.168.2.0/24
                                        -                     -       -       0       i

**vEOS-Leaf3**#sh ip route  vrf  TENANT100

        Gateway of last resort:
        B E      0.0.0.0/0 [20/0]
                via 10.10.10.2, Vlan4001

        C        10.10.10.0/30
                directly connected, Vlan4001
        B E      192.168.0.2/32 [20/0]
                via VTEP 20.255.255.1 VNI 111111 router-mac 50:d0:94:40:0d:78 local-interface Vxlan1
        B E      192.168.0.0/24 [20/0]
                via VTEP 20.255.255.1 VNI 111111 router-mac 50:d0:94:40:0d:78 local-interface Vxlan1

**vEOS-Leaf3**#sh ip route  vrf  TENANT200

        Gateway of last resort:
        B E      0.0.0.0/0 [20/0]
                via 20.20.20.2, Vlan4002

        C        20.20.20.0/30
                directly connected, Vlan4002
        B E      192.168.2.1/32 [20/0]
                via VTEP 20.255.255.2 VNI 222222 router-mac 50:27:14:56:01:70 local-interface Vxlan1
        B E      192.168.2.0/24 [20/0]
                via VTEP 20.255.255.2 VNI 222222 router-mac 50:27:14:56:01:70 local-interface Vxlan1

**vEOS-Leaf3**#sh bgp evpn route-type ip-prefix ipv4

        BGP routing table information for VRF default
        Router identifier 20.0.255.3, local AS number 64003
        Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                            c - Contributing to ECMP, % - Pending best path selection
        Origin codes: i - IGP, e - EGP, ? - incomplete
        AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

                Network                Next Hop              Metric  LocPref Weight  Path
        * >      RD: 20.0.255.3:100 ip-prefix 0.0.0.0/0
                                        -                     -       100     0       63000 i
        * >      RD: 20.0.255.3:200 ip-prefix 0.0.0.0/0
                                        -                     -       100     0       63000 i
        * >Ec    RD: 20.0.255.1:100 ip-prefix 192.168.0.0/24
                                        20.255.255.1          -       100     0       64000 64001 i
        *  ec    RD: 20.0.255.1:100 ip-prefix 192.168.0.0/24
                                        20.255.255.1          -       100     0       64000 64001 i
        * >Ec    RD: 20.0.255.2:200 ip-prefix 192.168.2.0/24
                                        20.255.255.2          -       100     0       64000 64002 i
        *  ec    RD: 20.0.255.2:200 ip-prefix 192.168.2.0/24
                                        20.255.255.2          -       100     0       64000 64002 i

#### Пинг между клиентам в разных VRF:

root@ubuntu:~# ping 192.168.2.1

        PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
        64 bytes from 192.168.2.1: icmp_seq=1 ttl=59 time=16.6 ms
        64 bytes from 192.168.2.1: icmp_seq=2 ttl=59 time=23.2 ms
        64 bytes from 192.168.2.1: icmp_seq=3 ttl=59 time=18.4 ms
        64 bytes from 192.168.2.1: icmp_seq=4 ttl=59 time=17.1 ms
        ^C
        --- 192.168.2.1 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3005ms
        rtt min/avg/max/mdev = 16.626/18.870/23.231/2.612 ms

#### Пинг наружу  Client1

root@ubuntu:~# ping 8.8.8.8  

        PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
        64 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=8.20 ms
        64 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=8.08 ms
        64 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=8.37 ms
        64 bytes from 8.8.8.8: icmp_seq=4 ttl=62 time=9.30 ms
        ^C
        --- 8.8.8.8 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3004ms
        rtt min/avg/max/mdev = 8.089/8.493/9.303/0.478 ms

#### Пинг наружу  Client3

root@ubuntu:~# ping 8.8.8.8

        PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
        64 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=8.74 ms
        64 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=9.15 ms
        64 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=19.8 ms
        64 bytes from 8.8.8.8: icmp_seq=4 ttl=62 time=9.28 ms
        ^C
        --- 8.8.8.8 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3004ms
        rtt min/avg/max/mdev = 8.742/11.762/19.868/4.685 ms