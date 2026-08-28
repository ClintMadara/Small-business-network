**Step 1 device layout**

Layout the devices on the in Packet tracer and connect them all using the copper straight through cable

- Web (Internet/Ecternal web server)
- Pc1  & PC2
- Switch
- Router 1 (acting as a pfsense firewall)
- Router 2 (acting as the default gateway)
- Server 1 (acting as the domain controller)
- Server 2 (acting as the database server)
- Server 3 (acting as the Splunk server)

**Step 2 - Cable connections**

- Web → pfSense Gi0/1
- pfSense Gi0/1 → Router0 Gi0/1
- Router0 Gi0/0 → Switch0 Gi0/1
- Switch0 Fa0/2 → PC0
- Switch0 Fa0/3 → PC1
- Switch0 Fa0/4 → Domain Controller
- Switch0 Fa0/5 → Database
- Switch0 Fa0/6 → Splunk

**Step 3 - Configuring switch and naming vlans**

Vlan 10 - Workstations

Vlan 20 - Servers

**Assiging the two PCs to vlan10**

interface FastEthernet0/1
switchport mode access
switchport access vlan 10

interface FastEthernet0/2
switchport mode access
switchport access vlan 10

---

**Assiging the servers to vlan20**

(Domain controller)
interface FastEthernet0/3
switchport mode access
switchport access vlan 20

(Database server)
interface FastEthernet0/4
switchport mode access
switchport access vlan 20

(Splunk server)
interface FastEthernet0/5
switchport mode access
switchport access vlan 20

---

**Configuring trunk on port switch (FastEthernet0/1)**

interface GigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 10,20

---

**Step 4 - Configuring (router0) subinterfaces for the vlans by splitting gigabitethernet0/0 into two interfaces**

- Gi0/0.10 = gateway for VLAN 10
- Gi0/0.20 = gateway for VLAN 20

interface GigabitEthernet0/0
no ip address
no shutdown

interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/1
no shutdown
end
write memory

---

**Step 5 - Setting static IPs on servers**

Domain Controller

IP: 192.168.20.10
Mask: 255.255.255.0
Gateway: 192.168.20.1
DNS: 192.168.20.10

</br>
<img width="1015" height="198" alt="image" src="https://github.com/user-attachments/assets/c54c5256-0980-4158-afde-0db76a2410ad" />
</br>
</br>
Database

IP: 192.168.20.11
Mask: 255.255.255.0
Gateway: 192.168.20.1
DNS: 192.168.20.10

</br>
<img width="1008" height="193" alt="image" src="https://github.com/user-attachments/assets/27f05fd0-50d6-4685-b0f1-e20fcd55de9b" />
</br>
</br>

Splunk

IP: 192.168.20.12
Mask: 255.255.255.0
Gateway: 192.168.20.1
DNS: 192.168.20.10

<img width="1010" height="199" alt="image" src="https://github.com/user-attachments/assets/281e361f-d521-4ecc-8920-a0f000da6b70" />

External Web server (Emulating the internet)

IP: 209.165.200.225
Mask: 255.255.255.252
Gateway: 209.165.200.226
DNS: 0

<img width="1003" height="178" alt="image" src="https://github.com/user-attachments/assets/3fb549bf-4cce-4faa-bffe-e168e41dc321" />

+ ping test - 192.168.20.10  to  192.168.20.12
<img width="449" height="189" alt="image" src="https://github.com/user-attachments/assets/098a572f-a89f-4eec-bc4c-d36f24da6b7c" />

+ ping test to default gateway 192.168.20.10  to  192.168.20.1
<img width="429" height="185" alt="image" src="https://github.com/user-attachments/assets/48c8d83f-a4dd-4e66-a06c-65aecd7c6d70" />

**Step 6 — Set up DNS on the Domain Controller**

- On the DC, go to services menu and find DNS tab
- Enable DNS
- Add the servers and their hostnames records

<img width="847" height="279" alt="image" src="https://github.com/user-attachments/assets/e1638759-0ced-4822-b38c-af7af927f2d8" />

+ Run an NSlookup on the DC to confirm that the IPs can be resolved and DNS is working

<img width="250" height="113" alt="image" src="https://github.com/user-attachments/assets/2168f92c-2997-47d7-bf93-56d07e11b7d6" />

---

**Step 7 — Set up DHCP for the workstations**

- On the DC, go to services menu and find DHCP tab
- Enable DHCP
- Name the pool to workstations
- Set default gateway to vlan 10 interface on the router: 192.168.10.1
- Set the DNS server to the IP on the Domain controller: 192.168.20.10
- Set start IP address to: 192.168.10.21
- Set the subnet mask to: 255.255.255.0

<img width="846" height="340" alt="image" src="https://github.com/user-attachments/assets/1315030c-377c-48bd-9d01-e561f50156ef" />

**Step 8 — Set up ACL to block workstations from accessing the database in vlan 20**

- Remove and delete any access control list assigned to the interace (GigabitEthernet0/0.10)

interface GigabitEthernet0/0.10
no ip access-group 110 in

no access-list 110

- Allow workstations to access only SQL on port 3306 on the database server but block everything else

access-list 110 permit tcp 192.168.10.0 0.0.0.255 host 192.168.20.11 eq 3306
access-list 110 deny ip 192.168.10.0 0.0.0.255 host 192.168.20.11

- Allow workstations to send logs to Splunk via their forwarder to the Splunk server on port 9907 and block everything else

access-list 110 permit tcp 192.168.10.0 0.0.0.255 host 192.168.20.12 eq 9997
access-list 110 deny ip 192.168.10.0 0.0.0.255 host 192.168.20.12

- Assign ACL 110 to the interface (GigabitEthernet0/0.10)

interface GigabitEthernet0/0.10
ip access-group 110 in

**Step 9 — Link pfSense router (firewall) to router**

- Configure router to send all **unknown traffic** to pfSense router (since the router only knows traffic from vlan 10 and vlan 20)

interface GigabitEthernet0/1
ip address 192.168.30.2 255.255.255.0
no shutdown
ip route 0.0.0.0 0.0.0.0 192.168.30.1

- Configure pfsense router to forward unknown traffic from router to the web server (The internet)

interface GigabitEthernet0/0
description WAN
ip address 209.165.200.226 255.255.255.252
no shutdown
ip route 0.0.0.0 0.0.0.0 209.165.200.225

<img width="926" height="691" alt="image" src="https://github.com/user-attachments/assets/87efadb3-18b5-4cfe-9b21-31da93fd15db" />

+ Ping test from router to pfSense (firewall): 192.168.30.1
<img width="507" height="95" alt="image" src="https://github.com/user-attachments/assets/12d4b14d-5dd4-4988-8427-411b8a5e1ae5" />

+ Ping test from router to external web server: 209.165.200.225
<img width="554" height="92" alt="image" src="https://github.com/user-attachments/assets/160d5366-0e4b-4e17-b74c-2680149b6f49" />

**Step 10 — Create ACL on pfsense to allow internal to external traffic as well as replies through the firewall but new external connections**

access-list 120 permit icmp any any echo-reply
access-list 120 permit tcp any any established
access-list 120 deny ip any any

interface GigabitEthernet0/0
ip access-group 120 in

- Ping test from PC0 to external web server: 192.168.10.21 to 209.165.200.225
<img width="466" height="194" alt="image" src="https://github.com/user-attachments/assets/a6590a67-bd8c-4fc5-a5e1-ea6c34751906" />

**Step 11 — Adding security to swithports by shutting down unused ports**

- Ports 1-2 are vlan 20
- Ports 3-5 are vlan 30
- Ports 6 and onwards are unused ports that need to be shutdown to reduce the threat surface

interface range FastEthernet0/6-24
shutdown
interface GigabitEthernet0/2
shutdown

<img width="556" height="388" alt="image" src="https://github.com/user-attachments/assets/2e5947e3-8efe-41ba-8157-6adb82438da0" />

**Step 12 — Port security**

- This is to make sure that port 1 to 5 are restricted to only one device (MAC address) to stop unknown devices from connecting to that port

interface range FastEthernet0/1-5
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
