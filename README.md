SECTION A (Hostname Username Password)
Synchronize the logs in all devices both for console and remote users
Turn off domain lookup in all devices 
Router0
router>enable
conf terminal
hostname router0
username router0 password router0
enable secret router0
service password encryption

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
conf terminal
no ip domain-lookup
end 
write

SECTION B	
1)Assign ip addresses
2)Configure access and trunk ports, assign vlans according to the picture

Vlan 45 - 10.49.21.160/28(255.255.255.240)
Gateway - 10.49.21.161
PC2 - 10.49.21.162
PC3 - 10.49.21.163
PC4 - 10.49.21.164

Server - 200.0.0.0/29(255.255.255.248)
Gateway - 200.0.0.1
DNS - 200.0.0.2
Khazar - 200.0.0.3

Router-Router - 10.0.0.0/30(255.255.255..252)
router0 - 10.0.0.1
router1 - 10.0.0.2
2)Switch vlan access trunk
switch0> enable
conf terminal
vlan 35
name VLAN35
exit
vlan 45
name VLan45
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
switchport trunk allowed vlan 35, 45
exit

3)Inter vlan routing
Configure DHCP for vlan 35, subnet-10.36.43.88/29, exclude the gateway IP address
router0>enable
conf terminal
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

4)Routerler arasi elaqe
router0>enable 
conf terminal
interface gig0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
exit

router1>enable
conf terminal
interface gig0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
exit

interface gig0/1
ip address 200.0.0.1 255.255.255.248
no shutdown
exit

5) static routing
router0>
ip route 200.0.0.0 255.255.255.248 10.0.0.2

router1>
ip route 10.36.43.88 255.255.255.248 10.0.0.1
ip route 10.49.21.160 255.255.255.240 10.0.0.1

6)DHCP
router0>
ip dhcp excluded address 10.36.43.89
ip dhcp pool vlan 35
network 10.36.43.88 255.255.255.248
default router 10.36.43.89
dns-server 200.0.0.2
exit

7)PC leri elle yaz
vlan 45 ve serverler

SONUNCU:
show vlan brief 
show interface trunk
show ip interface brief
show ip route


SECTION C
Configure OSPF
Loopback for router0-1.1.1.1
Loopback for router1-2.2.2.2

Configure the DNS server 
All the pC’s should be connecting to khazar server already

Activate ssh and telnet on both routers
Hostnames must be: router0 and router1, set domain name to khazar

router0>enable
conf terminal
interface loopback0
ip address 1.1.1.1 255.255.255.255
exit

router ospf 1
router-id 1.1.1.1

network 10.0.0.0 0.0.0.3 area 0
network 10.36.43.88 0.0.0.7 area 0
network 10.49.21.160 0.0.0.15 area 0
exit

Router1>enable
conf terminal
interface loopback0
ip address 2.2.2.2 255.255.255.255
exit

router ospf 1
router-id 2.2.2.2
network 10.0.0.0.0.0.0.3 area 0
network 200.0.0.0.0.0.0.7 area 0
exit

show ip ospf neighbor
show ip route ospf

2)Configure DNS server
Desktop -> IP configuration

Ip ->
Subnet mask ->
Default Gateway -> 

Services -> DNS
DNS -> On
name -> khazar
address -> 200.0.0.3

3)Active SSH & Telnet
Router0>enable
conf terminal
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

Router1>enable
conf terminal
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

ssh -1 admin 10.0.0.1
telnet 10.0.0.2


SECTION D: DYNAMIC NAT
Configure dynamic NAT for PC’s in vlan 45, translate IP address to 155.155.155.0/29 subnet

Router0>enable
conf terminal
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

show ip nat translations
show ip nat statistics

SECTION E:Configure access list
Deny only Web access for the first PC in vlan 45(PC2 in the picture) to khazar server 
Deny only Ping for the second PC in vlan 45 to khazar server 
Deny telnet connection for PC4 in vlan45 to router1
Allow everything else 

Xidmət	Protokol	Default Port
HTTP	TCP	80
HTTPS	TCP	443
FTP	TCP	21
SSH	TCP	22
TELNET	TCP	23
SMTP	TCP	25
DNS	UDP/TCP	53
PING	ICMP	port YOXDUR

ACL YARAT
router0>enable
configure terminal

ip access-list extended VLAN45_FILTER

deny tcp host 10.49.21.162 host 200.0.0.3 eq 80
deny tcp host 10.49.21.162 host 200.0.0.3 eq 443

deny icmp(ping) host 10.49.21.163 host 200.0.0.3 echo

deny tcp host 10.49.21.164 host 10.0.0.2 eq 23 
deny tcp host 10.49.21.164 host 200.0.0.1 eq 23

permit ip any any
exit

interface gig0/1.45
ip access-group VLAN45_FILTER in
exit

end
write


show access-lists(yoxlama)



