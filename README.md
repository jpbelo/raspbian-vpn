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
6. Connect to the Pi using SSH
	1. on OSX, open the terminal and type ``ssh pi@xxx.xxx.xxx.xxx`` (with the Pi's local IP address instead of the xxx). This may prompt a security question, just type ``yes`` and press enter.
	1. on Windows, use [PuTTY](http://www.putty.org)
7. When asked for the password, the raspbian default is ``raspberry``
8. The connection will be made and terminal will show ``pi@raspberry:~ $``. All the commands from now on, until we ``exit``, will run in the Pi.
9. Update the OS
	3.1. ``sudo apt-get update``
	3.2. ``sudo apt-get dist-upgrade``






## Configure VPN


2. procurar pelo IP do pi, usar app IP Scanner ou comando
			arp -a

3. ligar por ssh
			ssh pi@192.168.x.x
			raspberry

4. mudar password
			passwd

5. actualizar
			sudo apt-get update
			sudo apt-get upgrade

6. mudar o IP para estatico - ver imagens anexas para mais info (para sair " ctrl + X ")
			ifconfig
			sudo nano /etc/network/interfaces

									# interfaces(5) file used by ifup(8) and ifdown(8)

									# Please note that this file is written to be used with dhcpcd
									# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

									# Include files from /etc/network/interfaces.d:
									source-directory /etc/network/interfaces.d

									auto lo
									iface lo inet loopback

									auto eth0
									iface eth0 inet static
									address 192.168.173.8
									netmask 255.255.255.0
									network 192.168.173.0
									broadcast 192.168.173.255
									gateway 192.168.173.254
									dns-nameservers 193.137.55.20 193.137.55.10


7. instalar openVPN - dentro da pasta do user "pi" ( cd ~ )
			wget https://git.io/vpn -O openvpn-install.sh
			bash openvpn-install.ssh

8. criar mais certificados
			bash openvpn-install.ssh

9. para mover ficheiros:
	a) ir para o sitio ondem tem os ficheiros .ovpn (2 vezes para trás e entrar em root) pode ser preciso "sudo su"
	b) 		mv nomeficheiro.ovpn /home/pi/nomeficheiro.ovpn









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


