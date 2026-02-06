# Д/з №8
## VXLAN. Multihoming
Цель:
Реализовать передачу суммарных префиксов через EVPN route-type 5.
Сделать по возможности MULTIPOD

## Сводные таблицы изменены

## Сводная таблица Juniper
|  Оборудование  |      Lo0       |
| -------------- |----------------| 
|    Leaf1       | 10.0.255.1/32  | 
|    Leaf2       | 10.0.255.2/32  | 
|    Leaf3       | 10.0.255.3/32  | 
|    Spine1      | 10.0.0.1/32    |
|    Spine2      | 10.0.0.2/32    |
|    Client1     | 192.168.0.1/32 | 
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

## Сводная таблица BorderLeaf + EDGE router + Multipod

|        BorderLeaf        |        p2p       |     EDGE     |
|--------------------------|------------------|--------------|
|    ARISTA Leaf3 eth3     |  20.20.20.20/30  |   ge-0/0/0   |
|  Juniper Leaf1 xe-0/0/4  |  10.10.10.10/30  |   ge-0/0/1   |

|          BorderLeaf          |        p2p       |
|------------------------------|------------------|
|    ARISTA Leaf3 eth3  ip+2   |  На EGDE BRIDGE  |
|  Juniper Leaf1 xe-0/0/4 ip+1 |  30.30.30.30/30  |

## Схема
![Общая схема](https://raw.githubusercontent.com/wenter666/OTUS/refs/heads/main/lab7/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0.png)

## Сводная таблица AS Juniper
|  Оборудование  |      Lo0      |   AS  |
| -------------- |---------------|-------|
|    Leaf1       | 10.0.255.1/32 | 65001 |
|    Leaf2       | 10.0.255.2/32 | 65002 |
|    Leaf3       | 10.0.255.3/32 | 65003 |
|    Spine1      | 10.0.0.1/32   | 65000 |
|    Spine2      | 10.0.0.2/32   | 65000 |

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

    set protocols bgp group MULTIPOD export Export_Loopback_BGP
    set protocols bgp group MULTIPOD local-as 65001
    set protocols bgp group MULTIPOD neighbor 30.30.30.1 peer-as 64003
    set protocols bgp group MULTIPOD_EVPN-VxLAN multihop no-nexthop-change
    set protocols bgp group MULTIPOD_EVPN-VxLAN local-address 10.0.255.1
    set protocols bgp group MULTIPOD_EVPN-VxLAN family evpn signaling
    set protocols bgp group MULTIPOD_EVPN-VxLAN graceful-restart
    set protocols bgp group MULTIPOD_EVPN-VxLAN neighbor 20.0.255.3 peer-as 64003

### LEAF1 ARISTA

router bgp 64003
   router-id 20.0.255.3
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   maximum-paths 4 ecmp 64
   neighbor MULTIPOD peer group
   neighbor MULTIPOD remote-as 65001
   neighbor MULTIPOD_EVPN-VxLAN peer group
   neighbor MULTIPOD_EVPN-VxLAN remote-as 65000
   neighbor MULTIPOD_EVPN-VxLAN next-hop-unchanged
   neighbor MULTIPOD_EVPN-VxLAN update-source Loopback0
   neighbor MULTIPOD_EVPN-VxLAN ebgp-multihop 3
   neighbor MULTIPOD_EVPN-VxLAN send-community extended
   neighbor OVERLAY_EVPN-VxLAN peer group
   neighbor OVERLAY_EVPN-VxLAN remote-as 64000
   neighbor OVERLAY_EVPN-VxLAN update-source Loopback0
   neighbor OVERLAY_EVPN-VxLAN ebgp-multihop 3
   neighbor OVERLAY_EVPN-VxLAN send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 64000
   neighbor 10.0.255.1 peer group MULTIPOD_EVPN-VxLAN
   neighbor 20.0.0.1 peer group OVERLAY_EVPN-VxLAN
   neighbor 20.0.0.2 peer group OVERLAY_EVPN-VxLAN
   neighbor 20.0.1.5 peer group underlay
   neighbor 20.0.2.5 peer group underlay
   neighbor 30.30.30.2 peer group MULTIPOD
   !
   vlan 300
      rd 64003:300300
      route-target both 300:300300
      redistribute learned
   !
   address-family evpn
      neighbor MULTIPOD_EVPN-VxLAN activate
      neighbor OVERLAY_EVPN-VxLAN activate
   !
   address-family ipv4
      neighbor MULTIPOD activate
      neighbor underlay activate
      network 20.0.255.3/32
      network 20.255.255.3/32
   !
   vrf Intermediate_VNI
      rd 20.0.255.3:1
      route-target import evpn 1:1
      route-target export evpn 1:1
      redistribute connected
!

### BGP

root@vQFX-RE-Leaf1> show bgp summary 
  20.0.255.3            64003       3631       3554       0       1  1d 1:28:42 Establ
    Intermediate_VNI.evpn.0: 5/5/5/0
    __default_evpn__.evpn.0: 0/0/0/0
    bgp.evpn.0: 15/15/15/0
    default-switch.evpn.0: 10/10/10/0
  30.30.30.1            64003       3463       3275       0       3  1d 0:34:38 Establ
    inet.0: 8/8/8/0


    vEOS-Leaf3#sh bgp  summary 
    BGP summary information for VRF default
    Router identifier 20.0.255.3, local AS number 64003
    Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
    ---------- ----------- ------------- ----------------------- -------------- ---------- ----------
    10.0.255.1       65000 Established   L2VPN EVPN              Negotiated             30         30
    30.30.30.2       65001 Established   IPv4 Unicast            Negotiated              5          5


#### MAC/IP Client3 за Juniper, вывод сделан на ARISTA LEAF1 (делаем вывод что долетает)

    * >Ec    RD: 10.0.255.3:65000 mac-ip 300300 5047.e104.c400 192.168.2.1
                                    10.0.255.3            -       100     0       64000 64003 65000 i
    *  ec    RD: 10.0.255.3:65000 mac-ip 300300 5047.e104.c400 192.168.2.1
                                    10.0.255.3            -       100     0       64000 64003 65000 i

#### MAC/IP Client1 за ARISTA, вывод сделан на Juniper LEAF1 (делаем вывод что долетает)

    root@vQFX-RE-Spine2> show route | match 192.168.0.2 
    2:64001:100100::100100::50:2a:c5:04:86:00::192.168.0.2/304 MAC/IP        
    2:64002:100100::100100::50:2a:c5:04:86:00::192.168.0.2/304 MAC/IP  


#### Пинги от Lo0 ARISTA до Lo0 Juniper летят

vEOS-Leaf1# ping 10.0.255.3 source 20.0.255.1
PING 10.0.255.3 (10.0.255.3) from 20.0.255.1 : 72(100) bytes of data.
80 bytes from 10.0.255.3: icmp_seq=1 ttl=60 time=474 ms
80 bytes from 10.0.255.3: icmp_seq=2 ttl=60 time=465 ms
80 bytes from 10.0.255.3: icmp_seq=3 ttl=60 time=455 ms
80 bytes from 10.0.255.3: icmp_seq=4 ttl=60 time=445 ms
80 bytes from 10.0.255.3: icmp_seq=5 ttl=60 time=434 ms


root@vQFX-RE-Leaf3> ping 20.0.255.1 source 10.0.255.3 
PING 20.0.255.1 (20.0.255.1): 56 data bytes
64 bytes from 20.0.255.1: icmp_seq=0 ttl=60 time=376.898 ms
64 bytes from 20.0.255.1: icmp_seq=1 ttl=60 time=490.660 ms
64 bytes from 20.0.255.1: icmp_seq=2 ttl=60 time=500.144 ms
^C
--- 20.0.255.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 376.898/455.901/500.144/55.997 ms


Но не работает.

root@ubuntu:~# ping 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
^C
--- 192.168.2.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2010ms



Если брать один VNI между multihoming leaf1+leaf2 ARISTA и leaf1+leaf2 Juniper, все работает. То есть L3 работает, а вот L3 не хочет.

root@ubuntu:~# ping 192.168.0.1
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=242 ms
64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=157 ms
64 bytes from 192.168.0.1: icmp_seq=3 ttl=64 time=380 ms
^C

