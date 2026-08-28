**Step 1 device layout**

Layout the devices on the in Packet tracer and connect them all using the copper straight through cable

- Clout-Pt (Internet)
- Pc1  & PC2
- Switch
- Router 1 (acting as a pfsense firewall)
- Router 2 (acting as the default gateway)
- Server 1 (acting as the domain controller)
- Server 2 (acting as the database server)
- Server 3 (acting as the Splunk server)

**Step 2 - Cable connections**

- Clout-Pt Ethernet 6 → pfSense Gi0/1
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
