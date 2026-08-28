In step 6 of the configuration, DHCP failed after setting the workstations  to DHCP mode
</br>
</br>
<img width="806" height="43" alt="image" src="https://github.com/user-attachments/assets/2a444e25-23cf-4ca0-a1d5-cff02dac41e0" />
</br>
</br>
Reason
DHCP server (Domain controller) is on vlan20 and the workstations are on vlan10
</br>
Solution

Configure a DHCP relay on router

interface GigabitEthernet0/0.10
ip helper-address 192.168.20.10
</br>
</br>

<img width="1013" height="218" alt="image" src="https://github.com/user-attachments/assets/c3952ef0-f5e8-4e8b-a038-60f09136262b" />
