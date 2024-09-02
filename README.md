# How To Configuration Mikrotik RouterOS 6.49

Basic Configuration :
---------------
```
/system identity set name=[HOSTNAME]
```

Interface Configuration :
---------------
Configuration Loopback
```
/interface bridge add name=lo0
/ip address add address=[IP_ADDRESS/32] interface=lo0 network=[NETWORK_ADDRESS] comment=[DESCRIPTION]
```
Configuration Port Interface
```
/ip address add address=[IP_ADDRESS/30] interface=[PORT] network=[NETWORK_ADDRESS] comment=[DESCRIPTION]
/ip address add address=[IP_ADDRESS/30] interface=[PORT] network=[NETWORK_ADDRESS] comment=[DESCRIPTION]
/ip address add address=[IP_ADDRESS/30] interface=[PORT] network=[NETWORK_ADDRESS] comment=[DESCRIPTION]
```

Routing OSPF Configuration :
---------------
OSPF Interface
```
/routing ospf instance set default router-id=[IP_LOOPBACK]
/routing ospf interface add cost=[1-65000] interface=lo0 network-type=point-to-point passive=yes
/routing ospf interface add authentication=md5 authentication-key=mikrotik123 cost=[1-65000] dead-interval=10s hello-interval=5s interface=[PORT] network-type=point-to-point
/routing ospf interface add authentication=md5 authentication-key=mikrotik123 cost=[1-65000] dead-interval=10s hello-interval=5s interface=[PORT] network-type=point-to-point
/routing ospf interface add authentication=md5 authentication-key=mikrotik123 cost=[1-65000] dead-interval=10s hello-interval=5s interface=[PORT] network-type=point-to-point
```
OSPF Network Backbone Area
```
/routing ospf network add network=[NETWORK_ADDRESS] area=backbone <- AREA=0.0.0.0 
/routing ospf network add network=[NETWORK_ADDRESS] area=backbone
/routing ospf network add network=[NETWORK_ADDRESS] area=backbone
```
OSPF Network Multiarea
```
/routing ospf area add name=[OSPF_ID_A] area-id=[OSPF_ID_A]
/routing ospf area add name=[OSPF_ID_B] area-id=[OSPF_ID_B]
/routing ospf area add name=[OSPF_ID_C] area-id=[OSPF_ID_C]
/routing ospf network add network=[NETWORK_ADDRESS] area=backbone
/routing ospf network add network=[NETWORK_ADDRESS] area=[OSPF_ID_A]
/routing ospf network add network=[NETWORK_ADDRESS] area=[OSPF_ID_B]
/routing ospf network add network=[NETWORK_ADDRESS] area=[OSPF_ID_C]
```

MPLS LDP Configuration :
---------------
MPLS LDP and Advertise Neighbor
```
/mpls ldp interface add interface=[PORT]
/mpls ldp interface add interface=[PORT]
/mpls ldp interface add interface=[PORT]
/mpls ldp set enabled=yes lsr-id=[IP_LOOPBACK] transport-address=[IP_LOOPBACK]  <- MPLS NEIGHBOR
```

Routing BGP Configuration :
---------------
Configuration BGP to Route Reflector
```
/routing bgp instance set default as=[AS_NUMBER] router-id=[IP_LOOPBACK]
/routing bgp peer add remote-address=[IP_ROUTE_REFLECTOR] remote-as=[AS_NUMBER] address-families=vpnv4 update-source=lo0 name={DESCRIPTION] tcp-md5-key=[PASSWORD]
```
Configuration Route Reflector to RR Client
```
/routing bgp instance set default as=[AS_NUMBER] router-id=[IP_LOOPBACK]
/routing bgp peer add remote-address=[IP_RR_CLIENT_A] remote-as=[AS_NUMBER] address-families=vpnv4 update-source=lo0 route-reflect=yes name={DESCRIPTION] tcp-md5-key=[PASSWORD]
/routing bgp peer add remote-address=[IP_RR_CLIENT_B] remote-as=[AS_NUMBER] address-families=vpnv4 update-source=lo0 route-reflect=yes name={DESCRIPTION] tcp-md5-key=[PASSWORD]
/routing bgp peer add remote-address=[IP_RR_CLIENT_C] remote-as=[AS_NUMBER] address-families=vpnv4 update-source=lo0 route-reflect=yes name={DESCRIPTION] tcp-md5-key=[PASSWORD]
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
/interface bridge add name=VRF-AB mtu=1500 protocol-mode=rstp
/ip address add address=10.10.10.1/30 interface=VRF-AB
/interface bridge port add interface=ether3 bridge=VRF-AB bpdu-guard=yes
/ip route vrf add routing-mark=VRF-AB route-distinguisher=65000:2024 export-route-targets=65000:2024 import-route-targets=65000:2024 interfaces=VRF-AB
/routing bgp instance vrf add instance=default routing-mark=VRF-AB redistribute-connected=yes redistribute-static=yes 

@NODE-B
/interface bridge add name=VRF-AB mtu=1500 protocol-mode=rstp
/ip address add address=20.20.20.1/30 interface=VRF-AB
/interface bridge port add interface=ether3 bridge=VRF-AB bpdu-guard=yes
/ip route vrf add routing-mark=VRF-AB route-distinguisher=65000:2024 export-route-targets=65000:2024 import-route-targets=65000:2024 interfaces=VRF-AB
/routing bgp instance vrf add instance=default routing-mark=VRF-AB redistribute-connected=yes redistribute-static=yes 
```

ASBR Example Configuration :
---------------
Virtual Interface
```
/interface bonding add mode=802.3ad name=ae6 slaves=ether3 transmit-hash-policy=layer-2-and-3
```
IP Address
```
/ip address add address=105.0.0.1/30 comment=To-IX-10 interface=ether1-international network=105.0.0.0
/ip address add address=202.0.0.1/30 comment=To-IIX-32 interface=ether2-indonesia network=202.0.0.0
/ip address add address=110.0.0.1/24 comment=To-IP-Adv-110 interface=ae6 network=110.0.0.0
```
BGP Instance
```
/routing bgp instance set default disabled=yes
/routing bgp instance add as=110 name=EBGP-110 router-id=110.0.0.254
```
BGP Peering
```
/routing bgp peer add in-filter=international-in instance=EBGP-110 name=PEER-IX out-filter=international-out remote-address=105.0.0.2 remote-as=10
/routing bgp peer add in-filter=indonesia-in instance=EBGP-110 name=PEER-IIX out-filter=indonesia-out remote-address=202.0.0.2 remote-as=32
```
BGP Filter
```
/routing filter add action=accept chain=indonesia-out prefix=110.0.0.0/24
/routing filter add action=discard chain=indonesia-out
/routing filter add action=accept chain=international-out prefix=110.0.0.0/24 set-bgp-prepend=2
/routing filter add action=discard chain=international-out
/routing filter add action=accept chain=indonesia-in set-bgp-weight=101
/routing filter add action=accept chain=international-in set-bgp-weight=100
```
DNS 
```
/ip dns set allow-remote-requests=yes servers=110.0.0.1
```
Advertise BGP 
```
/routing bgp network add network=110.0.0.0/24 synchronize=no
```






