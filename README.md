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
4. Wait a few secunds for it to boot
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
	```
		# interfaces(5) file used by ifup(8) and ifdown(8)

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
		dns-nameservers 8.8.8.8 8.8.4.4
	```
5. press ``ctrl + x`` to exit, ``y`` to answer yes to the save prompt, ``enter`` to apply the changes to the file
6. restarting the network may suffice but I rather reboot the system - ``sudo reboot``
7. ssh again, this time with the new static local IP address and the new password.






## Configure VPN
1. instalar openVPN - dentro da pasta do user "pi" ( cd ~ )
			wget https://git.io/vpn -O openvpn-install.sh
			bash openvpn-install.ssh

2. criar mais certificados
			bash openvpn-install.ssh

3. para mover ficheiros:
	1. ir para o sitio ondem tem os ficheiros .ovpn (2 vezes para trás e entrar em root) pode ser preciso "sudo su"
	2. mv nomeficheiro.ovpn /home/pi/nomeficheiro.ovpn









FTP

1. instalar
			sudo apt-get install vsftpd

2. comandos
			service vsftpd start
			service vsftpd stop
			service vsftpd status

(tirar o passivo e usar porta 21)







FIREWALL - opcional
https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server

1. instalar
			sudo apt-get install ufw

2. meter yes no IPv6 (o default normalmente é yes)
			sudo vi /etc/default/ufw
			IPV6=yes

3. setup:
	permitir todas as saidas e nenhuma entrada:
			sudo ufw default deny incoming
			sudo ufw default allow outgoing

	permitir VPN
			sudo ufw allow 1194

	permitir SSH
			sudo ufw allow ssh
		ou (se a porta não for 22)
			sudo ufw allow 2222/tcp

	permitir FTP (e apagar)
			sudo ufw allow ftp
			sudo ufw delete allow ftp


	desliga e liga
		sudo ufw status
		sudo ufw disable
		sudo ufw enable







outros:

			sudo ufw allow www
			sudo ufw allow 80/tcp

			sudo ufw allow ftp
			sudo ufw allow 21/tcp


			sudo ufw delete allow ssh
			sudo ufw delete allow 80/tcp


		reset todos os settings
			sudo ufw reset


