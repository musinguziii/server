SSH
Device: Cisco Router or Switch
Switch(config)# hostname Switch1
Switch1(config)# ip domain-name example.com
Switch1(config)# crypto key generate rsa
(Choose key size: 1024 or 2048 bits)
Switch1(config)# username admin secret cisco123
Switch1(config)# line vty 0 4
Switch1(config-line)# transport input ssh
Switch1(config-line)# login local
Switch1(config-line)# exit
Switch1(config)# ip ssh version 2


VPN
Device: Cisco Router (on both ends)
Basic IPsec VPN setup between two sites:
! Phase 1
Router(config)# crypto isakmp policy 10
Router(config-isakmp)# encryption aes
Router(config-isakmp)# authentication pre-share
Router(config-isakmp)# group 2
Router(config-isakmp)# exit
Router(config)# crypto isakmp key vpnkey123 address 192.168.2.1

! Phase 2
Router(config)# crypto ipsec transform-set MYSET esp-aes esp-sha-hmac

! Crypto map
Router(config)# crypto map VPN-MAP 10 ipsec-isakmp
Router(config-crypto-map)# set peer 192.168.2.1
Router(config-crypto-map)# set transform-set MYSET
Router(config-crypto-map)# match address 100
Router(config)# access-list 100 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255

! Apply crypto map to outbound interface
Router(config)# interface g0/0
Router(config-if)# crypto map VPN-MAP


ACL
Device: Router or Switch (Layer 3)
Standard ACL to block host 192.168.1.100 from accessing the network:
Router(config)# access-list 10 deny 192.168.1.100
Router(config)# access-list 10 permit any
Router(config)# interface g0/0
Router(config-if)# ip access-group 10 in

Extended ACL to allow only HTTP traffic:
Router(config)# access-list 110 permit tcp any any eq 80
Router(config)# access-list 110 deny ip any any
Router(config)# interface g0/1
Router(config-if)# ip access-group 110 in


NAT
Device: Router
Basic PAT Configuration (Many-to-One):
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# ip nat inside

Router(config)# interface g0/1
Router(config-if)# ip address 203.0.113.1 255.255.255.0
Router(config-if)# ip nat outside

Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface g0/1 overload


DHCP
Router(config)# ip dhcp pool LAN
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10

OSPF
Router(config)# router ospf 1
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router(config-router)# network 10.0.0.0 0.255.255.255 area 0

Inter-VLAN Routing
Step 1: On the Switch (create VLANs)
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config)# vlan 20
Switch(config-vlan)# name HR

Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config)# interface fa0/24
Switch(config-if)# switchport mode trunk


On the Router (subinterfaces)
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface g0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
