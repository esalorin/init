Create new user and add sudo rights:
  * sudo adduser username
  * /sbin/adduser username sudo
  * sudo apt-get -u upgrade --assume-no
  
Check partitions:
  * sudo fdisk -l || lsblk
  
Make static IP:
  * make file to interfaces.d folder and add there:
  - iface enp0s3 inet static
  	address "your local IP address" u can choose the last numbers (10.12.180.73)
			net mask (\30) 255.255.255.252
				gateway	   (10.12.254.254)
						   (broadcast no needed)
  * AND modify /etc/network/interfaces file, remove last line (same than first line of the file u just created)
  * sudo systemctl restart networking
  * use command : ip link set enp0s3 up

Check open ports:
  * sudo nmap -sT -O localhost or ip
  OR nmap -p- ip
  
Change ssh port:
  * file /etc/ssh/sshd_config -> find line port 22
  * now choose your port and add it "port 55555" for example
  * to see internet network services list: grep ssh /etc/services
  * sudo ufw allow 55555/tcp (your port)
  * then restart ssh : sudo systemctl restart ssh (check sudo systemctl status ssh)
  * connect to ssh username@ip-addr -p port
  
SSH with public keys only:
  DO THIS IN ITERM
  * ssh-keygen
  * ssh-copy-id username@ip -p port
  (* in virtual box now change to /etc/ssh/sshd_config:
  	 - permitrootlogin: no
	 - passwordauthentication: no )
  * restart ssh
  * now log in frrom iterm with "ssh -p port username@ip"
  
The rules of your firewall: (UFW)
  * check status of ufw : sudo ufw status
  * enable ufw : sudo ufw enable ( to disable sudo ufw disable)
  * add rules to wirefaal : sudo ufw allow "portnum/service" (ssh, http, https)
  
Set DOS protection on your open ports :
  * install apt-get install fail2ban
  * cp file /etc/fail2ban/jail.conf -> jail.local
  * edit file: add jail to sshd, http-get-dos, apache?
  * with sudo fail2ban-client status u can see jail list
  * test for example open ssh session from iterm wth wrong user couple time and u get banned
  * u can see more iformation and if some ip is banned with sudo fail2ban-client status "service" 
