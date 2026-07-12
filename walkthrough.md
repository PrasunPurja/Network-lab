# Configuration Walkthrough

Key snippets from the full configs (in the configs folder), with explanations of what each
part does and why. This isn't the complete config but just the parts worth understanding.

## VLAN Creation (SW1)

*`vlan 10
name Sales`*

*`vlan 20
name HR`*

Creates two separate broadcast domains on the switch. Devices in VLAN 10 can't directly reach
devices in VLAN 20 at Layer 2. They need a router to communicate, which is what R1 is for.

## Trunk Port (SW1 → R1)
*`interface fastEthernet0/1`*

*`switchport mode trunk`*

By default, a switch port only carries one VLAN's traffic (an access port). A trunk port
instead carries *multiple* VLANs' traffic over a single physical link, tagging each frame with
its VLAN ID (802.1Q) so the receiving device knows which VLAN it belongs to. This is what lets
one physical cable to R1 carry both Sales and Engineering traffic.

## Router-on-a-Stick Sub-interfaces (R1)
*`interface gigabitEthernet0/0.10`*

*`encapsulation dot1Q 10`*

*`ip address 192.168.10.1 255.255.255.192`*

*`interface gigabitEthernet0/0.20`*

*`encapsulation dot1Q 20`*

*`ip address 192.168.10.65 255.255.255.224`*

A single physical router interface is split into two logical sub-interfaces, each tied to one
VLAN via `encapsulation dot1Q`. Each acts as that VLAN's gateway. This is how R1 routes
between VLANs using only one physical cable to the switch, instead of needing a separate cable
per VLAN.

## OSPF (R1 and R2)

*`router ospf 1`*

*`network 192.168.10.0 0.0.0.63 area 0`*

*`network 192.168.10.64 0.0.0.31 area 0`*

*`network 192.168.10.96 0.0.0.3 area 0`*

Rather than manually telling R1 and R2 about every remote network (static routes), OSPF lets
them discover each other's connected networks automatically. The numbers after each network
are wildcard masks (roughly the inverse of a subnet mask). They tell OSPF which interfaces
to advertise. Static routes were initially configured and verified working, then removed once
OSPF was confirmed to provide the same connectivity automatically.

## DHCP Pool (R1, Sales VLAN)

*`ip dhcp excluded-address 192.168.10.1`*

*`ip dhcp pool SALES_POOL`*

*`network 192.168.10.0 255.255.255.192`*

*`default-router 192.168.10.1`*

*`dns-server 192.168.10.10`*

Lets Sales-VLAN PCs receive an IP address, gateway, and DNS server automatically instead of
being configured manually. The `excluded-address` line prevents DHCP from accidentally handing
out R1's own gateway address to a PC.

## Extended ACL (R1, applied on Engineering's sub-interface, inbound)

*`access-list 101 permit ip 192.168.10.64 0.0.0.31 host 192.168.10.10`*

*`access-list 101 permit icmp 192.168.10.64 0.0.0.31 192.168.10.0 0.0.0.63 echo-reply`*

*`access-list 101 deny ip 192.168.10.64 0.0.0.31 192.168.10.0 0.0.0.63`*

*`access-list 101 permit ip any any`*

Implements a segmentation policy: HR is blocked from reaching Sales in general, with
two deliberate exceptions i.e. access to the shared DNS server, and ICMP echo-replies (needed
because ACLs are stateless and don't automatically allow return traffic for connections
initiated from Sales). Order matters here: exceptions must be listed before the general deny,
since ACLs are evaluated top-down and stop at the first matching line.

