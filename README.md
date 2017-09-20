# raspbian-vpn
Instructions for setting up a raspberry pi for vpn





## Before starting
### hardware
Raspberry Pi + power supply + SD card (min. 8 Gb)

### download the operating system
[Raspbian](https://www.raspberrypi.org/downloads/raspbian/) (the minimal version)

### download the software for writing the image to the SD card
[Etcher](https://etcher.io) (OSX) OR [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) (Windows)





## Install and setup OS
1. Use Etcher or Win32 Disk Imager to write the raspbian image to the SD card
2. Create an empty file with the name ``ssh`` (no extention) and place it on the sd root - this will enable ssh access to the pi
3. Put the SD in the Pi, connect it to the network and then the power supply
4. Wait a few seconds for it to boot
5. On a computer in the same network, look for the Pi's local IP address (I'm using the [IP Scanner](https://itunes.apple.com/pt/app/ip-scanner/id404167149?mt=12))
	1. ``arp -a`` on the terminal may also do the job
6. Connect to the Pi using SSH
	1. on OSX, open the terminal and type ``ssh pi@xxx.xxx.xxx.xxx`` (with the Pi's local IP address instead of the xxx). This may prompt a security question, just type ``yes`` and press enter.
	1. on Windows, use [PuTTY](http://www.putty.org)
7. When asked for the password, the raspbian default is ``raspberry``
8. The connection will be made and terminal will show ``pi@raspberry:~ $``. All the commands from now on, until we ``exit``, will run in the Pi.
9. Update the OS
	1. ``sudo apt-get update``
	2. ``sudo apt-get dist-upgrade``
10. To change the Pi login password: type ``passwd``, press enter, and when asked, the new password





## Make the Pi's local IP address static
1. ``ifconfig`` on the terminal to show the network information. What we need is in the ``eth0`` section. Make a note of the values:
	1. inet (current local IP address)
	2. netmask (mine was 255.255.255.0)
	3. broadcast (mine was 192.168.1.255)
2. ``route -n`` and take note of the Gateway (mine was 192.168.1.254)
3. ``cat /etc/resolv.conf`` and take note of the nameservers (DNS) used
4. Pick a local IP address for the Pi that has not been attributed to other machine - best pick from outside the automatic IP range (usually 50 and up) and the router IP (usually 1).
I'm using 8 in this exemple (you can use any IP but may need to reserve that IP on the router). Edit the interfaces file ``sudo nano /etc/network/interfaces`` (to exit the text editor press ``ctrl + x``) and edit it so it looks something like:
	```# interfaces(5) file used by ifup(8) and ifdown(8)

		# Please note that this file is written to be used with dhcpcd
		# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

		# Include files from /etc/network/interfaces.d:
		source-directory /etc/network/interfaces.d

		auto lo
		iface lo inet loopback

		auto eth0
		iface eth0 inet static
		address 192.168.1.8
		netmask 255.255.255.0
		network 192.168.1.0
		broadcast 192.168.1.255
		gateway 192.168.1.254
		dns-nameservers 8.8.8.8 8.8.4.4```
5. press ``ctrl + x`` to exit, ``y`` to answer yes to the save prompt, ``enter`` to apply the changes to the file
6. restarting the network may suffice but I rather reboot the system - ``sudo reboot``
7. ssh again, this time with the new static local IP address and the new password.





## Setup VPN
1. install openVPN - inside the user "pi" folder ( ``cd ~`` )
	1. ``wget https://git.io/vpn -O openvpn-install.sh``
	2. ``sudo bash openvpn-install.ssh``
2. follow the steps using the recommended values
3. we need to move the key certificate to a folder to later access it with FTP
	1. go to the folder where the .ovpn is stored. we need su permissions, so run ``sudo su`` and then ``cd ../../root``
	2. check what files are inside this folder with ``ls`` - there should be a .ovpn file inside with the name chosen during the openvpn install
	3. move the file to the pi user's folder with ``mv filename.ovpn /home/pi/filename.ovpn``
	4. use ``ls`` to check if the file is no longer here
	5. go to that folder ``cd ../home/pi`` OR ``cd ~``
	6. ``exit`` to leave su mode
	7. ``ls`` to check the file is now here
4. after install, everytime we need to create a new key certificates for a new user:
	1. from the pi user folder (``cd ~``) run the installer ``sudo bash openvpn-install.ssh``
	2. press ``1`` for new
	3. type the name for what machine this is going to be used on and press enter. The file will be generated inside the root folder.





## Setup FTP
1. install the FTP service with ``sudo apt-get install vsftpd``
2. check if the service is running with ``service vsftpd status``
3. on your computer, open any FTP application (Transmit, Cyberduck, FileZilla,...) and connect to the Pi FTP. The server will be the Pi's local IP, the user is ``pi`` and the password will be the one used on the Pi login
	1. I had to disable the option "use passive mode" and use port 21
4. copy the .ovpn file to your computer. This will be the key certificate to install on you device.
5. usefull commands for the FTP service:
	```
	service vsftpd start
	service vsftpd stop
	service vsftpd status
	```





## Install key on devices
For this stage I recomend using another device so you don't need to keep reconnecting to the Pi. I use the OpenVPN app on the iPhone. For OSX use [Tunnelblick](https://tunnelblick.net)

1. Install the key
	1. iOS - email yourself, open the file on the iPhone and chose to open it with the OpenVPN app
	2. OSX - just drag the file to the Tunnelblick top bar icon
2. Turn on the VPN
	1. iOS - inside the OpenVPN app, press the on/off switch. You can then just use the VPN switch inside the iOS settings
	2. OSX - click the top bar icon and chose connect "name_of_client"

A good wait to make sure it's working, is to use mobile data (turn wifi off) and go to [what is my ip](https://www.whatismyip.com) and check if your IP is the same as your home network.







## Firewall - optional
Best install this after VPN is working, so when it stops working, you know you need to tweek the firewall doors.

1. install the Uncomplicated Firewall with ``sudo apt-get install ufw``
2. edit the file ufw - ``sudo nano /etc/default/ufw``
	1. ``yes`` in the IPv6 (the default) - ``IPV6=yes``
	2. ``ctrl + x`` to exit
3. setup the firewall:
	1. to allow all outgoing and none incoming - ``sudo ufw default allow outgoing`` and ``sudo ufw default deny incoming``
	2. to allow VPN - ``sudo ufw allow 1194``
	3. to allow SSH - ``sudo ufw allow ssh`` or ``sudo ufw allow 2222/tcp`` (for specific port)
	4. to allow FTP - ``sudo ufw allow ftp`` (open when FTP is needed)
	5. to block FTP - ``sudo ufw delete allow ftp`` (close when FTP is not needed)
4. usefull ufw commands
	```
	sudo ufw status
	sudo ufw disable
	sudo ufw enable
	```
5. other commands:
	```
	sudo ufw allow www
	sudo ufw allow 80/tcp

	sudo ufw allow ftp
	sudo ufw allow 21/tcp

	sudo ufw delete allow ssh
	sudo ufw delete allow 80/tcp
	```
6. to reset all ufw settings ``sudo ufw reset``
