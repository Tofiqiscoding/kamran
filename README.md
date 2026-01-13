SECTION A – Hostname, Username, Password
Synchronize the logs in all devices (console and remote users)
Turn off domain lookup in all devices

Router0

router> enable
configure terminal

hostname router0
username router0 password router0
enable secret router0
service password-encryption

line console 0
login local
logging synchronous
exit

line vty 0 4
login local
logging synchronous
exit

end
write

configure terminal
no ip domain-lookup
end
write

SECTION B

Assign IP addresses

Configure access and trunk ports, assign VLANs according to the picture

VLAN 45 – 10.49.21.160/28 (255.255.255.240)
Gateway – 10.49.21.161
PC2 – 10.49.21.162
PC3 – 10.49.21.163
PC4 – 10.49.21.164

Server Network – 200.0.0.0/29 (255.255.255.248)
Gateway – 200.0.0.1
DNS – 200.0.0.2
Khazar – 200.0.0.3

Router to Router – 10.0.0.0/30 (255.255.255.252)
router0 – 10.0.0.1
router1 – 10.0.0.2

Switch VLAN Access & Trunk Configuration

switch0> enable
configure terminal

vlan 35
name VLAN35
exit

vlan 45
name VLAN45
exit

interface range fa0/2 - 3
switchport mode access
switchport access vlan 35
exit

interface range fa0/4 - 6
switchport mode access
switchport access vlan 45
exit

interface fa0/1
switchport mode trunk
switchport trunk allowed vlan 35,45
exit

Inter-VLAN Routing
Configure DHCP for VLAN 35
Subnet: 10.36.43.88/29
Exclude gateway IP address

router0> enable
configure terminal

interface gig0/1
no shutdown
exit

interface gig0/1.35
encapsulation dot1Q 35
ip address 10.36.43.89 255.255.255.248
exit

interface gig0/1.45
encapsulation dot1Q 45
ip address 10.49.21.161 255.255.255.240
exit

Routerlər arası əlaqə

router0> enable
configure terminal

interface gig0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
exit

router1> enable
configure terminal

interface gig0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
exit

interface gig0/1
ip address 200.0.0.1 255.255.255.248
no shutdown
exit

Static Routing

router0>
ip route 200.0.0.0 255.255.255.248 10.0.0.2

router1>
ip route 10.36.43.88 255.255.255.248 10.0.0.1
ip route 10.49.21.160 255.255.255.240 10.0.0.1

DHCP Configuration

router0>

ip dhcp excluded-address 10.36.43.89

ip dhcp pool VLAN35
network 10.36.43.88 255.255.255.248
default-router 10.36.43.89
dns-server 200.0.0.2
exit

PC-ləri manual IP ilə yaz
(VLAN 45 və Serverlər)

SONUNDA YOXLAMA KOMANDALARI

show vlan brief
show interface trunk
show ip interface brief
show ip route

SECTION C – OSPF, DNS, SSH & TELNET

Configure OSPF
Loopback router0 – 1.1.1.1
Loopback router1 – 2.2.2.2

router0> enable
configure terminal

interface loopback0
ip address 1.1.1.1 255.255.255.255
exit

router ospf 1
router-id 1.1.1.1
network 10.0.0.0 0.0.0.3 area 0
network 10.36.43.88 0.0.0.7 area 0
network 10.49.21.160 0.0.0.15 area 0
exit

router1> enable
configure terminal

interface loopback0
ip address 2.2.2.2 255.255.255.255
exit

router ospf 1
router-id 2.2.2.2
network 10.0.0.0 0.0.0.3 area 0
network 200.0.0.0 0.0.0.7 area 0
exit

OSPF yoxlama

show ip ospf neighbor
show ip route ospf

DNS Server Configuration

Desktop → IP Configuration

IP Address: 200.0.0.2
Subnet Mask: 255.255.255.248
Default Gateway: 200.0.0.1

Services → DNS

DNS: ON
Name: khazar
Address: 200.0.0.3

Activate SSH & Telnet

router0> enable
configure terminal

hostname router0
ip domain-name khazar
username admin password admin
enable secret admin

crypto key generate rsa
1024

line vty 0 4
login local
transport input ssh telnet
exit

router1> enable
configure terminal

hostname router1
ip domain-name khazar
username admin password admin
enable secret admin

crypto key generate rsa
1024

line vty 0 4
login local
transport input ssh telnet
exit

SSH & Telnet Test

ssh -l admin 10.0.0.1
telnet 10.0.0.2

SECTION D – Dynamic NAT

Dynamic NAT for VLAN 45 PCs
Public Subnet: 155.155.155.0/29

router0> enable
configure terminal

interface gig0/0
ip address 155.155.155.1 255.255.255.248
no shutdown
exit

interface gig0/1.45
ip nat inside
exit

interface gig0/0
ip nat outside
exit

ip nat pool VLAN45POOL 155.155.155.2 155.155.155.6 netmask 255.255.255.248

access-list 45 permit 10.49.21.160 0.0.0.15

ip nat inside source list 45 pool VLAN45POOL

NAT yoxlama

show ip nat translations
show ip nat statistics

SECTION E – Access Control List (ACL)

Məqsəd:

PC2 (VLAN 45) → Khazar Server üçün yalnız WEB (HTTP/HTTPS) qadağan

PC3 → Khazar Server üçün yalnız PING qadağan

PC4 → Router1 üçün TELNET qadağan

Qalan hər şeyə icazə

Protocol & Port Info

HTTP – TCP 80
HTTPS – TCP 443
FTP – TCP 21
SSH – TCP 22
TELNET – TCP 23
SMTP – TCP 25
DNS – TCP/UDP 53
PING – ICMP

ACL Configuration

router0> enable
configure terminal

ip access-list extended VLAN45_FILTER

deny tcp host 10.49.21.162 host 200.0.0.3 eq 80
deny tcp host 10.49.21.162 host 200.0.0.3 eq 443

deny icmp host 10.49.21.163 host 200.0.0.3 echo

deny tcp host 10.49.21.164 host 10.0.0.2 eq 23
deny tcp host 10.49.21.164 host 200.0.0.1 eq 23

permit ip any any
exit

interface gig0/1.45
ip access-group VLAN45_FILTER in
exit

end
write

ACL yoxlama

show access-lists
