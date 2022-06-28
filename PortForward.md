# How to Forward Ports on Your Router

Forwarding ports to the IP address of your LUKSO node can improve peer count. The steps for this process vary based on the manufacturer and model of your router.

You will need the following information for this guide:
* The IP address of your router
* The IP address of your node machine
* The username and password of your router

### Router IP Address

Run the following terminal command to find the IP address of your router:
```
netstat -nr | awk '$1 == "0.0.0.0"{print$2}'
```
### Node IP Address

Run the following terminal command to find the IP address of your router:
```
hostname -I
```

The output may display more than one address. The address you are looking for will begin with the same number as your router’s IP address, most likely 192.168.x.x  
Ignore any addresses that start with 172.


### Router Login


Enter the IP address of your router into the address bar of your web browser

A username and password prompt will appear. If you do now know your credential, you will need to reset your router to its default setting. Refer to your manual for instructions.

### Reserve Address of Node Machine

Address reservation ensures your node is always assigned the same IP address. The names of this setting can vary from router to router. It is often found under “DHCP Settings” or “DHCP Reservation.” Refer to your router’s manual for specific instructions.

Configure the router to reserve the node's current IP address (found in the Node [IP Address](#Node-IP-Address))

Make sure to click apply/save at the bottom of the page.

### Forward Ports

Steps for forwarding also vary by router brand.  

Common descriptions for these settings are “Port Forwarding” or “Virtual Server." Refer to your router’s manual for specific instructions.

Forward the following ports and protocols.

| Port      | Protocol | 
| -------   | -------- |
| 30303     | TCP      |
| 13000     | TCP
| 12000     | UDP      |
| 30303     | UDP      |

Enter the same number for *External Port* and *Internal Port*.
Enter the IP address of your node for the Internal/Destination address.
If prompted for a description, you may choose anything.


Make sure to click apply/save at the bottom of the page.

***This guide is community authored. It is NOT an official LUKSO document.***