# Финальное задание

Цель:
Настроить multipod между arista и juniper, применить технологии предыдущих уроков.

Описание:
Логика работы изменилась с предыдущих работ, теперь underlay OSPF, overlay EBGP.

Client1 существует в обоих POD, связность у него L2, так же в POD Juniper у него Multihoming.   
Связность с Client2 через L3VNI внутри POD Arista   
Связность с Client3 через L3VNI между POD Arista и POD Juniper 
Общий выход через EDGE роутер для обоих POD.    

Главное изменение вместо QFX, поставил MX960 с свежим софтом.

## Сводная таблица (Juniper)
|  Оборудование  |      Lo0       |   AS  |
| -------------- |----------------|-------| 
|    Leaf1       | 10.0.255.1/32  | 65001 |
|    Leaf2       | 10.0.255.2/32  | 65002 |
|    Leaf3       | 10.0.255.3/32  | 65003 | 
|    Spine1      | 10.0.0.1/32    | 65000 |
|    Spine2      | 10.0.0.2/32    | 65000 |
|    Client1     | 192.168.0.2/32 |   -   |
|    Client2     | 192.168.1.1/32 |   -   |


|   Spine ip+1   |      p2p      |   Leaf ip+0    |
|----------------|---------------|----------------|
|     Spine1     |               |                |
|    ge-0/0/1    |  10.0.1.0/31  | ge-0/0/1 leaf1 |
|    ge-0/0/2    |  10.0.1.2/31  | ge-0/0/1 leaf2 |    
|    ge-0/0/3    |  10.0.1.4/31  | ge-0/0/1 leaf3 |
|                |               |                |
|     Spine2     |               |                |
|    ge-0/0/1    |  10.0.2.0/31  | ge-0/0/2 leaf1 |
|    ge-0/0/2    |  10.0.2.2/31  | ge-0/0/2 leaf2 |    
|    ge-0/0/3    |  10.0.2.4/31  | ge-0/0/2 leaf3 |


## Сводная таблица (Arista)
|  Оборудование  |      Lo0       |   AS  |
| -------------- |----------------|-------| 
|    Leaf1       | 20.0.255.1/32  | 64001 |
|    Leaf2       | 20.0.255.2/32  | 64002 |
|    Leaf3       | 20.0.255.3/32  | 64003 | 
|    Spine1      | 20.0.0.1/32    | 64000 |
|    Spine2      | 20.0.0.2/32    | 64000 |
|    Client1     | 192.168.0.1/32 |   -   |
|    Client3     | 192.168.2.1/32 |   -   |


|   Spine ip+1   |      p2p      |   Leaf ip+0    |
|----------------|---------------|----------------|
|     Spine1     |               |                |
|      Eth1      |  20.0.1.0/31  |   Eth1 leaf1   |
|      Eth2      |  20.0.1.2/31  |   Eth1 leaf2   |    
|      Eth3      |  20.0.1.4/31  |   Eth1 leaf3   |
|                |               |                |
|     Spine2     |               |                |
|      Eth1      |  20.0.2.0/31  |   Eth2 leaf2   |
|      Eth2      |  20.0.2.2/31  |   Eth2 leaf2   |    
|      Eth3      |  20.0.2.4/31  |   Eth2 leaf2   |

## MULTIPOD
|                  |          p2p          |                   |
|------------------|-----------------------|-------------------|
|   Arista leaf1   | +1  10.10.10.0/31  +0 |   Juniper leaf1   |    
|       Eth3       |                       |      ge-0/0/0     |    

## EDGE

|                  |          p2p          |                   |
|------------------|-----------------------|-------------------|
|   Juniper leaf1  | +1  20.20.20.0/31 +0  |   Juniper EDGE    |    
|     ge-0/0/0     |                       |      ge-0/0/0     |    

## Схема
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/project/project.png)


## Выполнение, все настройки

### Juniper LEAF1

    set interfaces ge-0/0/0 description _To_Arista_LEAF3_eth3
    set interfaces ge-0/0/0 mtu 9014
    set interfaces ge-0/0/0 unit 0 description _To_Arista_LEAF3_eth3
    set interfaces ge-0/0/0 unit 0 family inet address 10.10.10.0/31
    set interfaces ge-0/0/1 description To-SPINE1
    set interfaces ge-0/0/1 mtu 9000
    set interfaces ge-0/0/1 unit 0 description To_SPINE1
    set interfaces ge-0/0/1 unit 0 family inet address 10.0.1.0/31
    set interfaces ge-0/0/2 description To-SPINE2
    set interfaces ge-0/0/2 mtu 9000
    set interfaces ge-0/0/2 unit 0 description To_SPINE2
    set interfaces ge-0/0/2 unit 0 family inet address 10.0.2.0/31
    set interfaces ge-0/0/3 description ae1-multihomig-to-client1
    set interfaces ge-0/0/3 gigether-options 802.3ad ae1
    set interfaces ae1 description SERVER_VLAN100_ACCESS
    set interfaces ae1 flexible-vlan-tagging
    set interfaces ae1 mtu 9000
    set interfaces ae1 encapsulation extended-vlan-bridge
    set interfaces ae1 esi 00:01:01:01:01:01:01:01:01:01
    set interfaces ae1 esi all-active
    set interfaces ae1 aggregated-ether-options lacp active
    set interfaces ae1 aggregated-ether-options lacp system-id 01:01:01:01:01:01
    set interfaces ae1 unit 100 vlan-id 100
    set interfaces irb unit 100 virtual-gateway-accept-data
    set interfaces irb unit 100 family inet address 192.168.0.100/24 virtual-gateway-address 192.168.0.254
    set interfaces irb unit 100 virtual-gateway-v4-mac 00:00:00:00:00:01
    set interfaces lo0 unit 0 family inet address 10.0.255.1/32
    set policy-options policy-statement LB term LB then load-balance per-flow
    set routing-instances Intermediate_VNI instance-type vrf
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes advertise direct-nexthop
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes encapsulation vxlan
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes vni 111111
    set routing-instances Intermediate_VNI vtep-source-interface lo0.0
    set routing-instances Intermediate_VNI interface irb.100
    set routing-instances Intermediate_VNI route-distinguisher 10.0.255.1:65001
    set routing-instances Intermediate_VNI vrf-target target:111:111
    set routing-instances evpn_vlan-based_vlan100 instance-type mac-vrf
    set routing-instances evpn_vlan-based_vlan100 protocols evpn encapsulation vxlan
    set routing-instances evpn_vlan-based_vlan100 protocols evpn default-gateway advertise
    set routing-instances evpn_vlan-based_vlan100 vtep-source-interface lo0.0
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 vlan-id 100
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 interface ae1.100
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 routing-interface irb.100
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 vxlan vni 100100
    set routing-instances evpn_vlan-based_vlan100 service-type vlan-based
    set routing-instances evpn_vlan-based_vlan100 route-distinguisher 65001:100100
    set routing-instances evpn_vlan-based_vlan100 vrf-target target:100:100100
    set routing-options router-id 10.0.255.1
    set routing-options autonomous-system 65001
    set routing-options forwarding-table export LB
    set protocols router-advertisement interface fxp0.0
    set protocols bgp group OVERLAY_EVPN-VxLAN type external
    set protocols bgp group OVERLAY_EVPN-VxLAN multihop no-nexthop-change
    set protocols bgp group OVERLAY_EVPN-VxLAN local-address 10.0.255.1
    set protocols bgp group OVERLAY_EVPN-VxLAN log-updown
    set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling
    set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart
    set protocols bgp group OVERLAY_EVPN-VxLAN multipath multiple-as
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.1 peer-as 65000
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.2 peer-as 65000
    set protocols bgp group JUNIPER_POD type external
    set protocols bgp group JUNIPER_POD multihop no-nexthop-change
    set protocols bgp group JUNIPER_POD local-address 10.0.255.1
    set protocols bgp group JUNIPER_POD log-updown
    set protocols bgp group JUNIPER_POD family evpn signaling
    set protocols bgp group JUNIPER_POD graceful-restart
    set protocols bgp group JUNIPER_POD multipath multiple-as
    set protocols bgp group JUNIPER_POD neighbor 20.0.255.3 peer-as 64003
    set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface lo0.0 passive
    set protocols ospf area 0.0.0.0 interface ge-0/0/0.0 interface-type p2p

### Juniper LEAF2

    set interfaces ge-0/0/0 mtu 9000
    set interfaces ge-0/0/1 description To-SPINE1
    set interfaces ge-0/0/1 mtu 9000
    set interfaces ge-0/0/1 unit 0 description To_SPINE1
    set interfaces ge-0/0/1 unit 0 family inet address 10.0.1.2/31
    set interfaces ge-0/0/2 description To-SPINE2
    set interfaces ge-0/0/2 mtu 9000
    set interfaces ge-0/0/2 unit 0 description To_SPINE2
    set interfaces ge-0/0/2 unit 0 family inet address 10.0.2.2/31
    set interfaces ge-0/0/3 description ae1-multihomig-to-client1
    set interfaces ge-0/0/3 gigether-options 802.3ad ae1
    set interfaces ae1 description SERVER_VLAN100_ACCESS
    set interfaces ae1 flexible-vlan-tagging
    set interfaces ae1 mtu 9000
    set interfaces ae1 encapsulation extended-vlan-bridge
    set interfaces ae1 esi 00:01:01:01:01:01:01:01:01:01
    set interfaces ae1 esi all-active
    set interfaces ae1 aggregated-ether-options lacp active
    set interfaces ae1 aggregated-ether-options lacp system-id 01:01:01:01:01:01
    set interfaces ae1 unit 100 vlan-id 100
    set interfaces irb unit 100 virtual-gateway-accept-data
    set interfaces irb unit 100 family inet address 192.168.0.101/24 virtual-gateway-address 192.168.0.254
    set interfaces irb unit 100 virtual-gateway-v4-mac 00:00:00:00:00:01
    set interfaces lo0 unit 0 family inet address 10.0.255.2/32
    set policy-options policy-statement LB term LB then load-balance per-flow
    set routing-instances Intermediate_VNI instance-type vrf
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes advertise direct-nexthop
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes encapsulation vxlan
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes vni 111111
    set routing-instances Intermediate_VNI vtep-source-interface lo0.0
    set routing-instances Intermediate_VNI interface irb.100
    set routing-instances Intermediate_VNI route-distinguisher 10.0.255.2:65002
    set routing-instances Intermediate_VNI vrf-target target:111:111
    set routing-instances evpn_vlan-based_vlan100 instance-type mac-vrf
    set routing-instances evpn_vlan-based_vlan100 protocols evpn encapsulation vxlan
    set routing-instances evpn_vlan-based_vlan100 protocols evpn default-gateway advertise
    set routing-instances evpn_vlan-based_vlan100 vtep-source-interface lo0.0
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 vlan-id 100
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 interface ae1.100
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 routing-interface irb.100
    set routing-instances evpn_vlan-based_vlan100 bridge-domains BD100 vxlan vni 100100
    set routing-instances evpn_vlan-based_vlan100 service-type vlan-based
    set routing-instances evpn_vlan-based_vlan100 route-distinguisher 65002:100100
    set routing-instances evpn_vlan-based_vlan100 vrf-target target:100:100100
    set routing-options router-id 10.0.255.2
    set routing-options autonomous-system 65002
    set routing-options forwarding-table export LB
    set protocols router-advertisement interface fxp0.0
    set protocols bgp group OVERLAY_EVPN-VxLAN type external
    set protocols bgp group OVERLAY_EVPN-VxLAN multihop
    set protocols bgp group OVERLAY_EVPN-VxLAN local-address 10.0.255.2
    set protocols bgp group OVERLAY_EVPN-VxLAN log-updown
    set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling
    set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart
    set protocols bgp group OVERLAY_EVPN-VxLAN multipath multiple-as
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.1 peer-as 65000
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.2 peer-as 65000
    set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface lo0.0 passive

### Juniper LEAF3
      
    set interfaces ge-0/0/0 description _To_EDGE_
    set interfaces ge-0/0/0 mtu 9000
    set interfaces ge-0/0/0 unit 0 description _To_EDGE_
    set interfaces ge-0/0/0 unit 0 family inet address 20.20.20.0/31
    set interfaces ge-0/0/1 description To_SPINE1
    set interfaces ge-0/0/1 mtu 9000
    set interfaces ge-0/0/1 unit 0 description To_SPINE1
    set interfaces ge-0/0/1 unit 0 family inet address 10.0.1.4/31
    set interfaces ge-0/0/2 description To_SPINE1
    set interfaces ge-0/0/2 mtu 9000
    set interfaces ge-0/0/2 unit 0 description To_SPINE2
    set interfaces ge-0/0/2 unit 0 family inet address 10.0.2.4/31
    set interfaces ge-0/0/3 description To_server_client2
    set interfaces ge-0/0/3 flexible-vlan-tagging
    set interfaces ge-0/0/3 native-vlan-id 300
    set interfaces ge-0/0/3 mtu 9000
    set interfaces ge-0/0/3 encapsulation extended-vlan-bridge
    set interfaces ge-0/0/3 unit 300 description To_server_client2_access_vlan300
    set interfaces ge-0/0/3 unit 300 vlan-id 300
    set interfaces irb unit 300 virtual-gateway-accept-data
    set interfaces irb unit 300 family inet address 192.168.1.100/24 virtual-gateway-address 192.168.1.254
    set interfaces irb unit 300 virtual-gateway-v4-mac 00:00:00:00:00:03
    set interfaces lo0 unit 0 family inet address 10.0.255.3/32
    set policy-options policy-statement EVPN-To-BGP_EDGE from route-filter 0.0.0.0/0 prefix-length-range /24-/24
    set policy-options policy-statement EVPN-To-BGP_EDGE then accept
    set policy-options policy-statement LB term LB then load-balance per-flow
    set routing-instances Intermediate_VNI instance-type vrf
    set routing-instances Intermediate_VNI protocols bgp group EBGP_EDGE export EVPN-To-BGP_EDGE
    set routing-instances Intermediate_VNI protocols bgp group EBGP_EDGE neighbor 20.20.20.1 peer-as 63000
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes advertise direct-nexthop
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes encapsulation vxlan
    set routing-instances Intermediate_VNI protocols evpn ip-prefix-routes vni 111111
    set routing-instances Intermediate_VNI vtep-source-interface lo0.0
    set routing-instances Intermediate_VNI interface ge-0/0/0.0
    set routing-instances Intermediate_VNI interface irb.300
    set routing-instances Intermediate_VNI route-distinguisher 10.0.255.3:65003
    set routing-instances Intermediate_VNI vrf-target target:111:111
    set routing-instances evpn_vlan-based_vlan300 instance-type mac-vrf
    set routing-instances evpn_vlan-based_vlan300 protocols evpn encapsulation vxlan
    set routing-instances evpn_vlan-based_vlan300 protocols evpn default-gateway advertise
    set routing-instances evpn_vlan-based_vlan300 vtep-source-interface lo0.0
    set routing-instances evpn_vlan-based_vlan300 bridge-domains BD300 vlan-id 300
    set routing-instances evpn_vlan-based_vlan300 bridge-domains BD300 interface ge-0/0/3.300
    set routing-instances evpn_vlan-based_vlan300 bridge-domains BD300 routing-interface irb.300
    set routing-instances evpn_vlan-based_vlan300 bridge-domains BD300 vxlan vni 300300
    set routing-instances evpn_vlan-based_vlan300 service-type vlan-based
    set routing-instances evpn_vlan-based_vlan300 route-distinguisher 65003:300300
    set routing-instances evpn_vlan-based_vlan300 vrf-target target:300:300300
    set routing-options router-id 10.0.255.3
    set routing-options autonomous-system 65003
    set routing-options forwarding-table export LB
    set protocols router-advertisement interface fxp0.0
    set protocols bgp group OVERLAY_EVPN-VxLAN type external
    set protocols bgp group OVERLAY_EVPN-VxLAN multihop
    set protocols bgp group OVERLAY_EVPN-VxLAN local-address 10.0.255.3
    set protocols bgp group OVERLAY_EVPN-VxLAN log-updown
    set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling
    set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart
    set protocols bgp group OVERLAY_EVPN-VxLAN multipath multiple-as
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.1 peer-as 65000
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.0.2 peer-as 65000
    set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface lo0.0 passive

### Juniper SPINE1

    set interfaces ge-0/0/0 mtu 9000
    set interfaces ge-0/0/1 description To-LEAF1
    set interfaces ge-0/0/1 mtu 9000
    set interfaces ge-0/0/1 unit 0 description To-LEAF1
    set interfaces ge-0/0/1 unit 0 family inet address 10.0.1.1/31
    set interfaces ge-0/0/2 description To-LEAF2
    set interfaces ge-0/0/2 mtu 9000
    set interfaces ge-0/0/2 unit 0 description To-LEAF2
    set interfaces ge-0/0/2 unit 0 family inet address 10.0.1.3/31
    set interfaces ge-0/0/3 description To-LEAF3
    set interfaces ge-0/0/3 mtu 9000
    set interfaces ge-0/0/3 unit 0 description To-LEAF3
    set interfaces ge-0/0/3 unit 0 family inet address 10.0.1.5/31
    set interfaces lo0 unit 0 family inet address 10.0.0.1/32
    set policy-options policy-statement LB term LB then load-balance per-flow
    set routing-options router-id 10.0.0.1
    set routing-options autonomous-system 65000
    set routing-options forwarding-table export LB
    set protocols router-advertisement interface fxp0.0
    set protocols bgp group OVERLAY_EVPN-VxLAN type external
    set protocols bgp group OVERLAY_EVPN-VxLAN multihop no-nexthop-change
    set protocols bgp group OVERLAY_EVPN-VxLAN local-address 10.0.0.1
    set protocols bgp group OVERLAY_EVPN-VxLAN log-updown
    set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling
    set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart
    set protocols bgp group OVERLAY_EVPN-VxLAN multipath multiple-as
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.255.1 peer-as 65001
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.255.2 peer-as 65002
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.255.3 peer-as 65003
    set protocols ospf area 0.0.0.0 interface lo0.0 passive
    set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/3.0 interface-type p2p

### Juniper SPINE2

    set interfaces ge-0/0/0 mtu 9000
    set interfaces ge-0/0/1 description To-LEAF1
    set interfaces ge-0/0/1 mtu 9000
    set interfaces ge-0/0/1 unit 0 description To-LEAF1
    set interfaces ge-0/0/1 unit 0 family inet address 10.0.2.1/31
    set interfaces ge-0/0/2 description To-LEAF2
    set interfaces ge-0/0/2 mtu 9000
    set interfaces ge-0/0/2 unit 0 description To-LEAF2
    set interfaces ge-0/0/2 unit 0 family inet address 10.0.2.3/31
    set interfaces ge-0/0/3 description To-LEAF3
    set interfaces ge-0/0/3 mtu 9000
    set interfaces ge-0/0/3 unit 0 description To-LEAF3
    set interfaces ge-0/0/3 unit 0 family inet address 10.0.2.5/31
    set interfaces lo0 unit 0 family inet address 10.0.0.2/32
    set policy-options policy-statement LB term LB then load-balance per-flow
    set routing-options router-id 10.0.0.2
    set routing-options autonomous-system 65000
    set routing-options forwarding-table export LB
    set protocols router-advertisement interface fxp0.0
    set protocols bgp group OVERLAY_EVPN-VxLAN type external
    set protocols bgp group OVERLAY_EVPN-VxLAN multihop no-nexthop-change
    set protocols bgp group OVERLAY_EVPN-VxLAN local-address 10.0.0.2
    set protocols bgp group OVERLAY_EVPN-VxLAN log-updown
    set protocols bgp group OVERLAY_EVPN-VxLAN family evpn signaling
    set protocols bgp group OVERLAY_EVPN-VxLAN graceful-restart
    set protocols bgp group OVERLAY_EVPN-VxLAN multipath multiple-as
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.255.1 peer-as 65001
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.255.2 peer-as 65002
    set protocols bgp group OVERLAY_EVPN-VxLAN neighbor 10.0.255.3 peer-as 65003
    set protocols ospf area 0.0.0.0 interface lo0.0 passive
    set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
    set protocols ospf area 0.0.0.0 interface ge-0/0/3.0 interface-type p2p

### Arista LEAF1

    hostname vEOS-Leaf1
    !
    spanning-tree mode mstp
    !
    system l1
      unsupported speed action error
      unsupported error-correction action error
    !
    vlan 100
      name vlan100
    !
    vrf instance Intermediate_VNI
    !
    interface Ethernet1
      description _To_SPINE1_
      mtu 9000
      no switchport
      ip address 20.0.1.0/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet2
      description _To_SPINE2_
      mtu 9000
      no switchport
      ip address 20.0.2.0/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet3
      mtu 9000
      switchport access vlan 100
    !
    interface Loopback0
      description MAIN
      ip address 20.0.255.1/32
      ip ospf area 0.0.0.0
    !
    interface Loopback1
      description FOR_VxLAN
      ip address 20.255.255.1/32
      ip ospf area 0.0.0.0
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
      vxlan vrf Intermediate_VNI vni 111111
      vxlan learn-restrict any
    !
    ip virtual-router mac-address 00:00:00:00:00:01
    !
    ip routing
    ip routing vrf Intermediate_VNI
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
      neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
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
          network 20.0.255.1/32
          network 20.255.255.1/32
      !
      vrf Intermediate_VNI
          rd 20.0.255.1:64001
          route-target import evpn 111:111
          route-target export evpn 111:111
          redistribute connected
    !
    router multicast
      ipv4
          software-forwarding kernel
      !
      ipv6
          software-forwarding kernel
    !
    router ospf 1
      router-id 20.0.255.1
      max-lsa 12000

### Arista LEAF2

    hostname vEOS-Leaf2
    !
    spanning-tree mode mstp
    !
    system l1
      unsupported speed action error
      unsupported error-correction action error
    !
    vlan 200
      name vlan200
    !
    vrf instance Intermediate_VNI
    !
    interface Ethernet1
      description _To_SPINE1_
      mtu 9000
      no switchport
      ip address 20.0.1.2/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet2
      description _To_SPINE2_
      mtu 9000
      no switchport
      ip address 20.0.2.2/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet3
      mtu 9000
      switchport access vlan 200
    !
    interface Loopback0
      description MAIN
      ip address 20.0.255.2/32
      ip ospf area 0.0.0.0
    !
    interface Loopback1
      description FOR_VxLAN
      ip address 20.255.255.2/32
      ip ospf area 0.0.0.0
    !
    interface Management1
    !
    interface Vlan200
      vrf Intermediate_VNI
      ip address virtual 192.168.2.254/24
    !
    interface Vxlan1
      vxlan source-interface Loopback1
      vxlan udp-port 4789
      vxlan vlan 200 vni 200200
      vxlan vrf Intermediate_VNI vni 111111
      vxlan learn-restrict any
    !
    ip virtual-router mac-address 00:00:00:00:00:02
    !
    ip routing
    ip routing vrf Intermediate_VNI
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
      neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
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
          network 20.0.255.2/32
          network 20.255.255.2/32
      !
      vrf Intermediate_VNI
          rd 20.0.255.2:64002
          route-target import evpn 111:111
          route-target export evpn 111:111
          redistribute connected
    !
    router multicast
      ipv4
          software-forwarding kernel
      !
      ipv6
          software-forwarding kernel
    !
    router ospf 1
      router-id 20.0.255.2
      max-lsa 12000

### Arista LEAF3

    hostname vEOS-Leaf3
    !
    spanning-tree mode mstp
    !
    system l1
      unsupported speed action error
      unsupported error-correction action error
    !
    vrf instance Intermediate_VNI
    !
    interface Ethernet1
      description _To_SPINE1_
      mtu 9000
      no switchport
      ip address 20.0.1.4/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet2
      description _To_SPINE2_
      mtu 9000
      no switchport
      ip address 20.0.2.4/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet3
      description _To_Juniper_LEAF1_
      mtu 9000
      no switchport
      ip address 10.10.10.1/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Loopback0
      description MAIN
      ip address 20.0.255.3/32
      ip ospf area 0.0.0.0
    !
    interface Loopback1
      description FOR_VxLAN
      ip address 20.255.255.3/32
      ip ospf area 0.0.0.0
    !
    interface Management1
    !
    interface Vxlan1
      vxlan source-interface Loopback1
      vxlan udp-port 4789
      vxlan learn-restrict any
    !
    ip routing
    ip routing vrf Intermediate_VNI
    !
    router bgp 64003
      router-id 20.0.255.3
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor JUNIPER_POD peer group
      neighbor JUNIPER_POD remote-as 65001
      neighbor JUNIPER_POD next-hop-unchanged
      neighbor JUNIPER_POD update-source Loopback0
      neighbor JUNIPER_POD ebgp-multihop 3
      neighbor JUNIPER_POD send-community extended
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN remote-as 64000
      neighbor OVERLAY_EVPN-VxLAN next-hop-unchanged
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor 10.0.255.1 peer group JUNIPER_POD
      neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
      !
      address-family evpn
          neighbor JUNIPER_POD activate
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          network 20.0.255.3/32
          network 20.255.255.3/32
      !
      vrf Intermediate_VNI
          rd 20.0.255.2:64003
          route-target import evpn 111:111
          route-target export evpn 111:111
          redistribute connected
    !
    router multicast
      ipv4
          software-forwarding kernel
      !
      ipv6
          software-forwarding kernel
    !
    router ospf 1
      router-id 20.0.255.3
      max-lsa 12000
    !

### Arista SPINE1

    hostname vEOS-Spine1
    !
    spanning-tree mode mstp
    !
    system l1
      unsupported speed action error
      unsupported error-correction action error
    !
    interface Ethernet1
      description _To_Leaf1_
      mtu 9000
      no switchport
      ip address 20.0.1.1/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet2
      description _To_Leaf2_
      mtu 9000
      no switchport
      ip address 20.0.1.3/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet3
      description _To_Leaf3_
      mtu 9000
      no switchport
      ip address 20.0.1.5/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Loopback0
      ip address 20.0.0.1/32
      ip ospf area 0.0.0.0
    !
    interface Management1
    !
    ip routing
    !
    router bgp 64000
      router-id 10.0.0.1
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN next-hop-unchanged
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor 20.0.255.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.255.1 remote-as 64001
      neighbor 20.0.255.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.255.2 remote-as 64002
      neighbor 20.0.255.3 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.255.3 remote-as 64003
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          no neighbor 20.0.1.0 activate
          no neighbor 20.0.1.2 activate
          no neighbor 20.0.1.4 activate
          network 20.0.0.1/32
    !
    router multicast
      ipv4
          software-forwarding kernel
      !
      ipv6
          software-forwarding kernel
    !
    router ospf 1
      router-id 20.0.0.1
      max-lsa 12000
    !

### Arista SPINE2

    hostname vEOS-Spine2
    !
    spanning-tree mode mstp
    !
    system l1
      unsupported speed action error
      unsupported error-correction action error
    !
    interface Ethernet1
      description _To_Leaf1_
      mtu 9000
      no switchport
      ip address 20.0.2.1/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet2
      description _To_Leaf2_
      mtu 9000
      no switchport
      ip address 20.0.2.3/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Ethernet3
      description _To_Leaf3_
      mtu 9000
      no switchport
      ip address 20.0.2.5/31
      ip ospf network point-to-point
      ip ospf area 0.0.0.0
    !
    interface Loopback0
      ip address 20.0.0.2/32
      ip ospf area 0.0.0.0
    !
    interface Management1
    !
    ip routing
    !
    router bgp 64000
      router-id 20.0.0.2
      no bgp default ipv4-unicast
      distance bgp 20 200 200
      maximum-paths 4 ecmp 64
      neighbor OVERLAY_EVPN-VxLAN peer group
      neighbor OVERLAY_EVPN-VxLAN next-hop-unchanged
      neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
      neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
      neighbor OVERLAY_EVPN-VxLAN send-community extended
      neighbor 20.0.255.1 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.255.1 remote-as 64001
      neighbor 20.0.255.2 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.255.2 remote-as 64002
      neighbor 20.0.255.3 peer group OVERLAY_EVPN-VxLAN
      neighbor 20.0.255.3 remote-as 64003
      !
      address-family evpn
          neighbor OVERLAY_EVPN-VxLAN activate
      !
      address-family ipv4
          no neighbor 20.0.2.0 activate
          no neighbor 20.0.2.2 activate
          no neighbor 20.0.2.4 activate
          network 20.0.0.2/32
    !
    router multicast
      ipv4
          software-forwarding kernel
      !
      ipv6
          software-forwarding kernel
    !
    router ospf 1
      router-id 20.0.0.2
      max-lsa 12000
    !

### Выводы EVPN маршрутов из arista LEAF 1/3

vEOS-**Leaf1**#sh bgp evpn 

    BGP routing table information for VRF default
    Router identifier 20.0.255.1, local AS number 64001
    Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                        c - Contributing to ECMP, % - Pending best path selection
    Origin codes: i - IGP, e - EGP, ? - incomplete
    AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

              Network                Next Hop              Metric  LocPref Weight  Path
    * >Ec    RD: 65001:100100 auto-discovery 0 0001:0101:0101:0101:0101
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 auto-discovery 0 0001:0101:0101:0101:0101
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65002:100100 auto-discovery 0 0001:0101:0101:0101:0101
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 auto-discovery 0 0001:0101:0101:0101:0101
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 10.0.255.1:0 auto-discovery 0001:0101:0101:0101:0101
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 10.0.255.1:0 auto-discovery 0001:0101:0101:0101:0101
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 10.0.255.2:0 auto-discovery 0001:0101:0101:0101:0101
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 10.0.255.2:0 auto-discovery 0001:0101:0101:0101:0101
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 10.0.255.1:0 auto-discovery 0500:00fd:e900:0187:0400
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 10.0.255.1:0 auto-discovery 0500:00fd:e900:0187:0400
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 10.0.255.2:0 auto-discovery 0500:00fd:ea00:0187:0400
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 10.0.255.2:0 auto-discovery 0500:00fd:ea00:0187:0400
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 10.0.255.3:0 auto-discovery 0500:00fd:eb00:0495:0c00
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 10.0.255.3:0 auto-discovery 0500:00fd:eb00:0495:0c00
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65001:100100 mac-ip 0000.0000.0001
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 mac-ip 0000.0000.0001
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65002:100100 mac-ip 0000.0000.0001
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 mac-ip 0000.0000.0001
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65001:100100 mac-ip 0000.0000.0001 192.168.0.254
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 mac-ip 0000.0000.0001 192.168.0.254
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65002:100100 mac-ip 0000.0000.0001 192.168.0.254
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 mac-ip 0000.0000.0001 192.168.0.254
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65003:300300 mac-ip 0000.0000.0003
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 0000.0000.0003
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65003:300300 mac-ip 0000.0000.0003 192.168.1.254
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 0000.0000.0003 192.168.1.254
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65003:300300 mac-ip 2c6b.f5a5.e3f0
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 2c6b.f5a5.e3f0
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65003:300300 mac-ip 2c6b.f5a5.e3f0 192.168.1.100
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 2c6b.f5a5.e3f0 192.168.1.100
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65002:100100 mac-ip 2c6b.f5a7.58f0
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 mac-ip 2c6b.f5a7.58f0
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65002:100100 mac-ip 2c6b.f5a7.58f0 192.168.0.101
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 mac-ip 2c6b.f5a7.58f0 192.168.0.101
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65001:100100 mac-ip 2c6b.f5fa.39f0
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 mac-ip 2c6b.f5fa.39f0
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65001:100100 mac-ip 2c6b.f5fa.39f0 192.168.0.100
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 mac-ip 2c6b.f5fa.39f0 192.168.0.100
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >      RD: 64001:100100 mac-ip 502a.c504.8600
                                    -                     -       -       0       i
    * >      RD: 64001:100100 mac-ip 502a.c504.8600 192.168.0.1
                                    -                     -       -       0       i
    * >Ec    RD: 64002:200200 mac-ip 5047.e104.c400
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 64002:200200 mac-ip 5047.e104.c400
                                    20.255.255.2          -       100     0       64000 64002 i
    * >Ec    RD: 64002:200200 mac-ip 5047.e104.c400 192.168.2.1
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 64002:200200 mac-ip 5047.e104.c400 192.168.2.1
                                    20.255.255.2          -       100     0       64000 64002 i
    * >Ec    RD: 65003:300300 mac-ip 505e.3205.3e00
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 505e.3205.3e00
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65003:300300 mac-ip 505e.3205.3e00 192.168.1.1
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 mac-ip 505e.3205.3e00 192.168.1.1
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 65001:100100 mac-ip 5066.1b04.c100
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 mac-ip 5066.1b04.c100
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65002:100100 mac-ip 5066.1b04.c100
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 mac-ip 5066.1b04.c100
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65001:100100 mac-ip 5066.1b04.c100 192.168.0.2
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 mac-ip 5066.1b04.c100 192.168.0.2
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65002:100100 mac-ip 5066.1b04.c100 192.168.0.2
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 mac-ip 5066.1b04.c100 192.168.0.2
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65001:100100 imet 10.0.255.1
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 65001:100100 imet 10.0.255.1
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 65002:100100 imet 10.0.255.2
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 65002:100100 imet 10.0.255.2
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 65003:300300 imet 10.0.255.3
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 65003:300300 imet 10.0.255.3
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >      RD: 64001:100100 imet 20.255.255.1
                                    -                     -       -       0       i
    * >Ec    RD: 64002:200200 imet 20.255.255.2
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 64002:200200 imet 20.255.255.2
                                    20.255.255.2          -       100     0       64000 64002 i
    * >Ec    RD: 10.0.255.1:0 ethernet-segment 0001:0101:0101:0101:0101 10.0.255.1
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 10.0.255.1:0 ethernet-segment 0001:0101:0101:0101:0101 10.0.255.1
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 10.0.255.2:0 ethernet-segment 0001:0101:0101:0101:0101 10.0.255.2
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 10.0.255.2:0 ethernet-segment 0001:0101:0101:0101:0101 10.0.255.2
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >Ec    RD: 10.0.255.3:65003 ip-prefix 0.0.0.0/0
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 63000 i
    *  ec    RD: 10.0.255.3:65003 ip-prefix 0.0.0.0/0
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 63000 i
    * >Ec    RD: 10.0.255.3:65003 ip-prefix 20.20.20.0/31
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 10.0.255.3:65003 ip-prefix 20.20.20.0/31
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 10.0.255.1:65001 ip-prefix 192.168.0.0/24
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    *  ec    RD: 10.0.255.1:65001 ip-prefix 192.168.0.0/24
                                    10.0.255.1            -       100     0       64000 64003 65001 i
    * >Ec    RD: 10.0.255.2:65002 ip-prefix 192.168.0.0/24
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    *  ec    RD: 10.0.255.2:65002 ip-prefix 192.168.0.0/24
                                    10.0.255.2            -       100     0       64000 64003 65001 65000 65002 i
    * >      RD: 20.0.255.1:64001 ip-prefix 192.168.0.0/24
                                    -                     -       -       0       i
    * >Ec    RD: 10.0.255.3:65003 ip-prefix 192.168.1.0/24
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    *  ec    RD: 10.0.255.3:65003 ip-prefix 192.168.1.0/24
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 i
    * >Ec    RD: 20.0.255.2:64002 ip-prefix 192.168.2.0/24
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 20.0.255.2:64002 ip-prefix 192.168.2.0/24
                                    20.255.255.2          -       100     0       64000 64002 i

vEOS-**Leaf3****#sh bgp evpn

    BGP routing table information for VRF default
    Router identifier 20.0.255.3, local AS number 64003
    Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                        c - Contributing to ECMP, % - Pending best path selection
    Origin codes: i - IGP, e - EGP, ? - incomplete
    AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

              Network                Next Hop              Metric  LocPref Weight  Path
    * >      RD: 65001:100100 auto-discovery 0 0001:0101:0101:0101:0101
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65002:100100 auto-discovery 0 0001:0101:0101:0101:0101
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 10.0.255.1:0 auto-discovery 0001:0101:0101:0101:0101
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 10.0.255.2:0 auto-discovery 0001:0101:0101:0101:0101
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 10.0.255.1:0 auto-discovery 0500:00fd:e900:0187:0400
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 10.0.255.2:0 auto-discovery 0500:00fd:ea00:0187:0400
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 10.0.255.3:0 auto-discovery 0500:00fd:eb00:0495:0c00
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65001:100100 mac-ip 0000.0000.0001
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65002:100100 mac-ip 0000.0000.0001
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65001:100100 mac-ip 0000.0000.0001 192.168.0.254
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65002:100100 mac-ip 0000.0000.0001 192.168.0.254
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65003:300300 mac-ip 0000.0000.0003
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65003:300300 mac-ip 0000.0000.0003 192.168.1.254
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65003:300300 mac-ip 2c6b.f5a5.e3f0
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65003:300300 mac-ip 2c6b.f5a5.e3f0 192.168.1.100
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65002:100100 mac-ip 2c6b.f5a7.58f0
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65002:100100 mac-ip 2c6b.f5a7.58f0 192.168.0.101
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65001:100100 mac-ip 2c6b.f5fa.39f0
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65001:100100 mac-ip 2c6b.f5fa.39f0 192.168.0.100
                                    10.0.255.1            -       100     0       65001 i
    * >Ec    RD: 64001:100100 mac-ip 502a.c504.8600
                                    20.255.255.1          -       100     0       64000 64001 i
    *  ec    RD: 64001:100100 mac-ip 502a.c504.8600
                                    20.255.255.1          -       100     0       64000 64001 i
    * >Ec    RD: 64001:100100 mac-ip 502a.c504.8600 192.168.0.1
                                    20.255.255.1          -       100     0       64000 64001 i
    *  ec    RD: 64001:100100 mac-ip 502a.c504.8600 192.168.0.1
                                    20.255.255.1          -       100     0       64000 64001 i
    * >Ec    RD: 64002:200200 mac-ip 5047.e104.c400
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 64002:200200 mac-ip 5047.e104.c400
                                    20.255.255.2          -       100     0       64000 64002 i
    * >Ec    RD: 64002:200200 mac-ip 5047.e104.c400 192.168.2.1
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 64002:200200 mac-ip 5047.e104.c400 192.168.2.1
                                    20.255.255.2          -       100     0       64000 64002 i
    * >      RD: 65003:300300 mac-ip 505e.3205.3e00
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65003:300300 mac-ip 505e.3205.3e00 192.168.1.1
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 65001:100100 mac-ip 5066.1b04.c100
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65002:100100 mac-ip 5066.1b04.c100
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65001:100100 mac-ip 5066.1b04.c100 192.168.0.2
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65002:100100 mac-ip 5066.1b04.c100 192.168.0.2
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65001:100100 imet 10.0.255.1
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 65002:100100 imet 10.0.255.2
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 65003:300300 imet 10.0.255.3
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >Ec    RD: 64001:100100 imet 20.255.255.1
                                    20.255.255.1          -       100     0       64000 64001 i
    *  ec    RD: 64001:100100 imet 20.255.255.1
                                    20.255.255.1          -       100     0       64000 64001 i
    * >Ec    RD: 64002:200200 imet 20.255.255.2
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 64002:200200 imet 20.255.255.2
                                    20.255.255.2          -       100     0       64000 64002 i
    * >      RD: 10.0.255.1:0 ethernet-segment 0001:0101:0101:0101:0101 10.0.255.1
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 10.0.255.2:0 ethernet-segment 0001:0101:0101:0101:0101 10.0.255.2
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >      RD: 10.0.255.3:65003 ip-prefix 0.0.0.0/0
                                    10.0.255.3            -       100     0       65001 65000 65003 63000 i
    * >      RD: 10.0.255.3:65003 ip-prefix 20.20.20.0/31
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >      RD: 10.0.255.1:65001 ip-prefix 192.168.0.0/24
                                    10.0.255.1            -       100     0       65001 i
    * >      RD: 10.0.255.2:65002 ip-prefix 192.168.0.0/24
                                    10.0.255.2            -       100     0       65001 65000 65002 i
    * >Ec    RD: 20.0.255.1:64001 ip-prefix 192.168.0.0/24
                                    20.255.255.1          -       100     0       64000 64001 i
    *  ec    RD: 20.0.255.1:64001 ip-prefix 192.168.0.0/24
                                    20.255.255.1          -       100     0       64000 64001 i
    * >      RD: 10.0.255.3:65003 ip-prefix 192.168.1.0/24
                                    10.0.255.3            -       100     0       65001 65000 65003 i
    * >Ec    RD: 20.0.255.2:64002 ip-prefix 192.168.2.0/24
                                    20.255.255.2          -       100     0       64000 64002 i
    *  ec    RD: 20.0.255.2:64002 ip-prefix 192.168.2.0/24
                                    20.255.255.2          -       100     0       64000 64002 i     


### Выводы Multihoming Juniper LEAF1/2 (DF LEAF1)

root@LEAF1> show evpn instance designated-forwarder 

    Instance: evpn_vlan-based_vlan100
      Number of ethernet segments: 3
        ESI: 00:01:01:01:01:01:01:01:01:01
          Designated forwarder: 10.0.255.1
        ESI: 05:00:00:fd:e9:00:01:87:04:00
        ESI: 05:00:00:fd:ea:00:01:87:04:00

root@LEAF1> show evpn instance esi-info 

    Instance: __default_evpn__

    Instance: evpn_vlan-based_vlan100
      Number of ethernet segments: 3
        ESI: 00:01:01:01:01:01:01:01:01:01
          Status: Resolved by IFL ae1.100
          Local interface: ae1.100, Status: Up/Forwarding
          Number of remote PEs connected: 1
            Remote-PE        MAC-label  Aliasing-label  Mode
            10.0.255.2       100100     100100          all-active   
          DF Election Algorithm: MOD based
          Designated forwarder: 10.0.255.1
          Backup forwarder: 10.0.255.2
          Last designated forwarder update: Feb 26 17:34:02
        ESI: 05:00:00:fd:e9:00:01:87:04:00
          Local interface: irb.100, Status: Up/Forwarding
        ESI: 05:00:00:fd:ea:00:01:87:04:00
          Status: Resolved
          Number of remote PEs connected: 1
            Remote-PE        MAC-label  Aliasing-label  Mode
            10.0.255.2       100100     0               all-active  

### L2VNI для Client1, L3 для CLient 2/3

vEOS-Leaf1#sh vxlan vni 

    VNI to VLAN Mapping for Vxlan1
    VNI          VLAN       Source       Interface       802.1Q Tag
    ------------ ---------- ------------ --------------- ----------
    100100       100        static       Ethernet3       untagged  
                                        Vxlan1          100       

    VNI to dynamic VLAN Mapping for Vxlan1
    VNI          VLAN       VRF                    Source       
    ------------ ---------- ---------------------- ------------ 
    111111       4101       Intermediate_VNI       evpn         

#### Дефолт долетает до крайней железки.

vEOS-Leaf1#sh bgp evpn route-type ip-prefix ipv4

              Network                Next Hop              Metric  LocPref Weight  Path
    * >Ec    RD: 10.0.255.3:65003 ip-prefix 0.0.0.0/0
                                    10.0.255.3            -       100     0       64000 64003 65001 65000 65003 63000 i


#### Пинг Client1 (POD ARISTA) ->  Client1 (POD Juniper)

root@ubuntu:~# ping 192.168.0.2

    PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
    64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=8.27 ms
    64 bytes from 192.168.0.2: icmp_seq=2 ttl=64 time=16.1 ms
    64 bytes from 192.168.0.2: icmp_seq=3 ttl=64 time=8.89 ms

#### Пинг Client1 (POD ARISTA) ->  Client3 (POD ARISTA)

root@ubuntu:~# ping 192.168.2.1

    PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
    64 bytes from 192.168.2.1: icmp_seq=1 ttl=62 time=13.3 ms
    64 bytes from 192.168.2.1: icmp_seq=2 ttl=62 time=8.44 ms
    64 bytes from 192.168.2.1: icmp_seq=3 ttl=62 time=7.46 ms

#### Пинг Client1 (POD ARISTA) ->  Client2 (POD Juniper)

root@ubuntu:~# ping 192.168.1.1

      PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
      64 bytes from 192.168.1.1: icmp_seq=1 ttl=62 time=11.8 ms
      64 bytes from 192.168.1.1: icmp_seq=2 ttl=62 time=10.8 ms
      64 bytes from 192.168.1.1: icmp_seq=3 ttl=62 time=16.6 ms

#### Пинг Client1 (POD ARISTA) ->  EDGE 8.8.8.8

root@ubuntu:~# ping 8.8.8.8

      PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
      64 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=10.6 ms
      64 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=11.5 ms
      64 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=10.8 ms         

#### Пинг Client3 (POD ARISTA) ->  EDGE 8.8.8.8                  

root@ubuntu:~#  ping 8.8.8.8

      PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
      64 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=30.3 ms
      64 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=21.1 ms
      64 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=13.1 ms

#### Пинг Client1 (POD JUNIPER) ->  EDGE 8.8.8.8

root@cli:~# ping 8.8.8.8

      PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
      64 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=5.97 ms
      64 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=5.52 ms
      64 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=5.64 ms
    
#### Пинг Client2 (POD JUNIPER) ->  EDGE 8.8.8.8

root@ubuntu:~#  ping 8.8.8.8

      PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
      64 bytes from 8.8.8.8: icmp_seq=1 ttl=63 time=2.17 ms
      64 bytes from 8.8.8.8: icmp_seq=2 ttl=63 time=2.01 ms
      64 bytes from 8.8.8.8: icmp_seq=3 ttl=63 time=2.42 ms
