# How To Configuration Mikrotik RouterOS 6.49

Basic Configuration :
---------------
```
/system identity set name=PE4
```

Interface Configuration :
---------------
Configuration Loopback
```
/interface bridge add name=lo0
/ip address add address=40.40.40.40 interface=lo0 network=40.40.40.40 comment=PE4
```
Configuration Port Interface
```
/ip address add address=172.0.0.22/30 interface=ether1 network=172.0.0.20 comment=to-PE2
/ip address add address=172.0.0.33/30 interface=ether2 network=172.0.0.32 comment=to-CE2
/ip address add address=172.0.0.18/30 interface=ether3 network=172.0.0.16 comment=to-P2
```

Routing OSPF Configuration :
---------------
OSPF Interface
```
/routing ospf instance set default router-id=40.40.40.40
/routing ospf interface add cost=65000 interface=lo0 network-type=point-to-point passive=yes
/routing ospf interface add authentication=md5 authentication-key=mikrotik123 cost=1 dead-interval=10s hello-interval=5s interface=ether1 network-type=point-to-point
/routing ospf interface add authentication=md5 authentication-key=mikrotik123 cost=1 dead-interval=10s hello-interval=5s interface=ether2 network-type=point-to-point
/routing ospf interface add authentication=md5 authentication-key=mikrotik123 cost=1 dead-interval=10s hello-interval=5s interface=ether3 network-type=point-to-point
```
OSPF Network Backbone Area
```
/routing ospf network add network=40.40.40.40/32 area=backbone
/routing ospf network add network=172.0.0.20/30 area=backbone
/routing ospf network add network=172.0.0.16/30 area=backbone
```
OSPF Network Multiarea
```
/routing ospf area add name=area20 area-id=0.0.0.20
/routing ospf network add network=172.0.0.32/30 area=area20
```

MPLS LDP Configuration :
---------------
MPLS LDP and Advertise Neighbor
```
/mpls ldp interface add interface=ether1
/mpls ldp interface add interface=ether2
/mpls ldp interface add interface=ether3
/mpls ldp set enabled=yes lsr-id=40.40.40.40 transport-address=40.40.40.40
```

Routing BGP Configuration :
---------------
Configuration BGP to Route Reflector
```
/routing bgp instance set default as=65000 router-id=40.40.40.40
/routing bgp peer add remote-address=192.168.0.254 remote-as=65000 address-families=vpnv4 update-source=lo0 name=to-RR tcp-md5-key=mikrotik123
```
Configuration Route Reflector to RR Client
```
/routing bgp instance set default as=65000 router-id=192.168.0.254
/routing bgp peer add remote-address=192.168.0.1 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-CE1 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=192.168.0.2 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-CE2 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=1.1.1.1 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-P1 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=2.2.2.2 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-P2 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=10.10.10.10 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-PE1 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=20.20.20.20 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-PE2 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=30.30.30.30 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-PE3 tcp-md5-key=mikrotik123
/routing bgp peer add remote-address=40.40.40.40 remote-as=65000 address-families=vpnv4 update-source=lo0 route-reflect=yes name=to-PE4 tcp-md5-key=mikrotik123
```

MPLS Service :
---------------
VPLS Configuration
```
Example:
@NODE-A
/interface vpls add disabled=no name=vpls-l2-circuit-2024 pw-type=tagged-ethernet remote-peer=192.168.0.2 vpls-id=2024:0
/interface bridge add mtu=1500 name=l2-circuit-2024
/interface bridge port add bridge=l2-circuit-2024 interface=ether4 bpdu-guard=yes 
/interface bridge port add bridge=l2-circuit-2024 interface=vpls-l2-circuit-2024 horizon=1

@NODE-B
/interface vpls add disabled=no name=vpls-l2-circuit-2024 pw-type=tagged-ethernet remote-peer=192.168.0.1 vpls-id=2024:0
/interface bridge add mtu=1500 name=l2-circuit-2024
/interface bridge port add bridge=l2-circuit-2024 interface=ether4 bpdu-guard=yes 
/interface bridge port add bridge=l2-circuit-2024 interface=vpls-l2-circuit-2024 horizon=1
```
MPLS L3VPN Configuration
```
Example:
@NODE-A
/system identity set name=CE1
/interface bridge add name=VRF-AB mtu=1500 protocol-mode=rstp
/ip address add address=10.10.10.1/30 interface=VRF-AB
/interface bridge port add interface=ether3 bridge=VRF-AB bpdu-guard=yes
/ip route vrf add routing-mark=VRF-AB route-distinguisher=65000:2024 export-route-targets=65000:2024 import-route-targets=65000:2024 interfaces=VRF-AB
/routing bgp instance vrf add instance=default routing-mark=VRF-AB redistribute-connected=yes redistribute-static=yes 

@NODE-B
/system identity set name=CE2
/interface bridge add name=VRF-AB mtu=1500 protocol-mode=rstp
/ip address add address=20.20.20.1/30 interface=VRF-AB
/interface bridge port add interface=ether3 bridge=VRF-AB bpdu-guard=yes
/ip route vrf add routing-mark=VRF-AB route-distinguisher=65000:2024 export-route-targets=65000:2024 import-route-targets=65000:2024 interfaces=VRF-AB
/routing bgp instance vrf add instance=default routing-mark=VRF-AB redistribute-connected=yes redistribute-static=yes 
```






