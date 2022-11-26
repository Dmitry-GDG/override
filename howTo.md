| [README](README.md) | [subject](subject_ru.md) | How to start a web server | [defense](defense.md) |
|-|-|-|-|

# How to start a web server

1. Download [the image](https://cdn.intra.42.fr/isos/OverRide.iso)
2. Go to that folder and convert the downloaded image to a format supported by VirtualBox:
```
VBoxManage convertfromraw --format VDI OverRide.iso OverRide.vdi

```

3. Run VirtualBox
	- click "New" button
	
	<img width="62" alt="Screen Shot 2022-11-11 at 10 24 40" src="https://user-images.githubusercontent.com/84193980/201290196-43410f66-660a-4540-9496-4bf91d85679a.png">

	- choose folder "Machine folder" with enough disk space (on schools Macs - goinfre)
	- Go to th "Expert mode"
	
	<img width="697" alt="Screen Shot 2022-11-26 at 12 41 14" src="https://user-images.githubusercontent.com/84193980/204082490-2f7fa2a1-d457-4caf-97e2-5e787598fc68.png">

	- Choose "Use an existing virtual hard disk drive" -> Click "Add" button
	
	<img width="64" alt="Screen Shot 2022-11-11 at 10 34 41" src="https://user-images.githubusercontent.com/84193980/201290340-7b50b622-ff19-4ec3-b8c1-f2405667ffa6.png">

	- choose Darkly_i386.vdi -> "Open" -> "Choose" 
	- fill "Name" field (override) and type "Create".

4. **Create a new host-only adapter in VirtualBox**. Create the Virtual Network. First, you must set up a virtual network that the host-only adapter(s) will communicate through.

	- In the VirtualBox window, click File -> Host Network Manager -> Create.

	- Check Enable under the DHCP Server column of the network you just created.

	- Select your network and click Properties.

	- In the Adapter Tab, select Configure Adapter Manually and use the following settings:
	```
	IPv4 Address: 192.168.56.1
	IPv4 Network Mask: 255.255.255.0
	```
	- In the DHCP Server Tab, make sure that Enable Server box is checked, and use the following settings:
	```
	Server Address: 192.168.56.100
	Server Mask: 255.255.255.0
	Lower Address Bound: 192.168.56.3
	Upper Address Bound: 192.168.56.254
	```
	- Click Apply and then Close

5. **Add a Host-Only Adapter to the Guest Machine.** For each guest you want to communicate with using the network from the previous step, you need to add a host-only adapter.

	- Select the appropriate guest machine (override)

	- Click Settings -> Аudio
	```
	Enable Audio: Unchecked
	```
	
	<img width="649" alt="Screen Shot 2022-11-26 at 12 32 24" src="https://user-images.githubusercontent.com/84193980/204082499-7378cbd3-5580-42ed-aadf-b35e29143eb5.png">

	- Click Settings -> Network

	- Under the Adapter tab, input the following settings:
	```
	Enable Network Adapter: Checked
	Attached to: Host-Only Adapter
	Name: vboxnet0 (NOTE: this should be the name of the network you created in the previous steps)
	In "Promiscuous Mode" select "Allow VMs"
	```

	<img width="649" alt="Screen Shot 2022-11-26 at 12 29 27" src="https://user-images.githubusercontent.com/84193980/204082505-1b8cacb1-0f34-4301-9e02-9157b4740139.png">

	- Click OK

	The host-only adapter should be ready to use on this machine. 
	```
	If you would like to network multiple machines together, repeat steps this paragraph 5 for each guest machine.
	```

6. Take shapshot of your VM

7. Launch VM. At the top venu your VM choose "View" -> "Virtual screen 1" -> "Scale to 275%" and get the IP address of the website to be audited 

<img width="1200" alt="Screen Shot 2022-11-11 at 11 16 21" src="https://user-images.githubusercontent.com/84193980/201297810-9f5e8917-2f55-4ee0-96ad-2d347395af7f.png">

8. That's all. Next step you will do on your Mac: type this IP address into your browser (?) and audit the website. Good luck!

<img width="1276" alt="Screen Shot 2022-11-11 at 11 23 36" src="https://user-images.githubusercontent.com/84193980/201297804-96514ae7-6e1f-46de-b987-7e2d2c776cee.png">


| [README](README.md) | [subject](subject_ru.md) | How to start a web server | [defense](defense.md) |
|-|-|-|-|
