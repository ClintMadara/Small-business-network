**In step 6 of the configuration, DHCP failed after setting the workstations  to DHCP mode**
</br>
</br>
<img width="995" height="38" alt="image" src="https://github.com/user-attachments/assets/69e0ac2a-c093-461b-8e06-2369f7709bde" />
</br>
### **Reason**

DHCP server (Domain controller) is on vlan20 and the workstations are on vlan10
</br>
Solution

### **Fix**

Configure a DHCP relay on router

interface GigabitEthernet0/0.10
ip helper-address 192.168.20.10
</br>
</br>

<img width="1013" height="218" alt="image" src="https://github.com/user-attachments/assets/c3952ef0-f5e8-4e8b-a038-60f09136262b" />
