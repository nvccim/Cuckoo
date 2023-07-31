## Requirements 

- Ubuntu 18.04 1G/60GB (install it with cuckoo as username)

		Install it with cuckoo as username

  		https://releases.ubuntu.com/18.04/

- windows 7 32bits

 		https://telecharger.malekal.com/download/iso-windows-7-pro-32-bits/

- Enable virtualisation Engine on ubuntu VM (Settings > Processor )

## Installation

- Install required packages 

		sudo apt-get -y install python python-pip python-dev libffi-dev libssl-dev python-virtualenv python-setuptools libjpeg-dev zlib1g-dev swig mongodb postgresql libpq-dev virtualbox tcpdump apparmor-utils

- Configure tcpdump 

		sudo aa-disable /usr/sbin/tcpdump

		sudo adduser --disabled-password --gecos "" cuckootest

		sudo groupadd pcap

		sudo usermod -a -G pcap cuckootest

		sudo chgrp pcap /usr/sbin/tcpdump

		sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

- Install M2Crypto:

    		sudo pip install m2crypto

- Add the account you created before to vboxusers group:

    		sudo usermod -a -G vboxusers cuckootest

- Create a virtual environment by using a script :

		chmod +x cuckoo-env.sh
		./cuckoo-env.sh
		source ~/.bashrc

- Create a virtual environment. You can name your virtual environment anything you want, I am using sandbox:

    	mkvirtualenv -p python2.7 sandbox

- Setup and install cuckoo while you are inside your newly created virtual env (sandbox) :

		pip install -U pip setuptools
		pip install -U pip cuckoo

- Install this packages again to make sure that there no missing package

		sudo apt-get -y install build-essential libssl-dev libffi-dev python-dev genisoimage zlib1g-dev libjpeg-dev python-pip python-virtualenv python-setuptools swig

## Setup the VirtualBox and its networking

- Create "Host-Only Adapter" by running the following command:
  
		vboxmanage hostonlyif create

 - Set the IP address for the vboxnet0 interface which you created before.

		vboxmanage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1

 - Open virtualbox from terminal on sandbox envirenment 

		Create windows 7 with 1G RAM / 25G ROM and named wincuckoo
		complete installation (Verify Network settings Attached To: *Host-Only-Adapter* || Name : *vboxnet0*) 	

- Configure Firewall :

		sudo sysctl -w net.ipv4.conf.vboxnet0.forwarding=1
		sudo sysctl -w net.ipv4.conf.ens33.forwarding=1
		sudo iptables -t nat -A POSTROUTING -o *your network adapter name* -s 192.168.56.0/24 -j MASQUERADE
 		sudo iptables -P FORWARD DROP
  		sudo iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
		sudo iptables -A FORWARD -s 192.168.56.0/24 -j ACCEPT

- Enable IP forwarding in the kernel. To that, you have to execute the following commands:

		echo 1 | sudo tee -a /proc/sys/net/ipv4/ip_forward
		sudo sysctl -w net.ipv4.ip_forward=1

## Setting up the Guest machine :

Setup manually adapter : (open network settings > modify network adapter > proprities > IPVv4 double click)

	IP Address — 192.168.56.101 (VM IP address)
	Subnet Mask — 255.255.255.0
	Default Gateway — 192.168.56.1 (Internet accessing interface)
	DNS Servers — 8.8.8.8/8.8.4.4

  	Disable Windows Update (Control panel > System andSecurity > disable firewall)
  	Disable Windows Firewall (Control panel > System and Security > Modify parametres > never search for updates)
  	Disable UAC (Windows + R > regedit > HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System > Modify EnableUA to 0)

## Setup python :

- Donwload :
  
		https://www.python.org/ftp/python/2.7.13/python-2.7.13.msi
  
- Setup web server :
  
		cd Downloads/
		python -m SimpleHTTPServer 8080

- Back to windows 7 :
  
		Browse http://IP:8080 
		Download python and install it

## Setup agent.py :

	(sandbox) cuckoo@ubuntu:~$ cuckoo
	(sandbox) cuckoo@ubuntu:~$ cd .cuckoo/agent
	(sandbox) cuckoo@ubuntu:~./cuckoo/agent$ zip agent.zip agent.py
	(sandbox) cuckoo@ubuntu:~./cuckoo/agent$ python -m SimpleHTTPServer 8080

- Back to windows 7 :

		Browse http://IP:8080 
		Download agent.zip
		uznip it and put agent.py on desktop	

## Setup snapshot :

	Open windows 7
	Open cmd as Administrator:
		cd C:\Python27\Scripts
		pip install pillow
  		C:\Python27\python.exe C:\PATH\OF\AGENT.PY

	VM > Machine > Take Snapshot > Name : snapsnap > Ok
	windows buttoon > shutdown the machine



## Setup Cuckoo :
  
- Open 3 terminal with sandbox envirenment :

		cuckoo@ubuntu:~$ workon sandbox
  
- First Terminal :
	
		(sandbox) cuckoo@ubuntu:~$ cd .cuckoo/conf/

- Second Terminal :

		(sandbox) cuckoo@ubuntu:~$ cuckoo rooter --sudo --group cuckoo

- Back to First terminal :
    
		edit routing.conf :
		internet = ens33

  		edit reporting.conf :
    
		[mongdb]
		enabled = yes

		edit virtualbox.conf :
		Delete anythings between controlports = 5000-5050 and [honeyd]
		Modify :
			machines = wincuckoo
		Add this between controlports = 5000-5050 and [honeyd]
			[wincuckoo]
			label = wincuckoo
			platform = windows
			ip = 192.168.56.101
			snapshot = snapsnap

- Open third Terminal :
  
		run cuckoo :
	   	(sandbox) cuckoo@ubuntu:~$ cuckoo

- Open fourth Terminal :
  
	   	(sandbox) cuckoo@ubuntu:~$ cuckoo web runserver

## Done !!
