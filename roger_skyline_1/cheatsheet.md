# Create new user and add sudo rights:
  * sudo adduser username
  * /sbin/adduser username sudo
  
# Check if all packages are updaed:
  * sudo apt-get update && apt-get -s upgrade ( -s will do a simulated upgrade showing packets that would be installed but will not actually install anything.
  
# Check partitions:
  * sudo fdisk -l || lsblk
  
# Make static IP:
  * make file to interfaces.d folder and add there:
  * iface enp0s3 inet static
  	- address "your local IP address" u can choose the last numbers (10.12.180.73)
	- net mask (\30) 255.255.255.252
	- gateway  (10.12.254.254)	   
	- (broadcast no needed)
  * AND modify /etc/network/interfaces file, remove last line (same than first line of the file u just created)
  * sudo systemctl restart networking
  * use command : ip link set enp0s3 up

# Check open ports:
  * sudo nmap -sT -O localhost or ip
  OR nmap -p- ip
  
# Change ssh port:
  * file /etc/ssh/sshd_config -> find line port 22
  * now choose your port and add it "port 55555" for example
  * to see internet network services list: grep ssh /etc/services
  * sudo ufw allow 55555/tcp (your port)
  * then restart ssh : sudo systemctl restart ssh (check sudo systemctl status ssh)
  * connect to ssh username@ip-addr -p port
  
# SSH with public keys only:
  DO THIS IN ITERM
  * ssh-keygen
  * ssh-copy-id username@ip -p port
  * (in virtual box now change to /etc/ssh/sshd_config:
  	 - permitrootlogin: no
	 - passwordauthentication: no )
  * restart ssh
  * now log in frrom iterm with "ssh -p port username@ip"
  
# The rules of your firewall: (UFW)
  * check status of ufw : sudo ufw status
  * enable ufw : sudo ufw enable ( to disable sudo ufw disable)
  * add rules to wirefaal : sudo ufw allow "portnum/service" (ssh, http, https)
  
# Set DOS protection on your open ports :
  * install apt-get install fail2ban
  * cp file /etc/fail2ban/jail.conf -> jail.local
  * edit file: add jail to sshd, http-get-dos, apache?
  	- here is example for http-get-dos:
		- [http-get-dos]
		- enabled = true
		- port = http,https
		- filter = http-get-dos
		- logpath = /var/log/varnish/access.log
		- maxretry = 300
		- findtime = 300
		- bantime = 600 (10 min)
		- action = iptables[name=HTTP, port=http, protocol=tcp]
	- you also need to make file /etc/fail2ban/filter.d/http-get-dos.conf
  * with sudo fail2ban-client status u can see jail list
  * test for example open ssh session from iterm wth wrong user couple time and u get banned
  * u can see more iformation and if some ip is banned with sudo fail2ban-client status "service" 

# Set protection against scanning ports :
  * u can scan all ports with nmap -p- <ip>
  * install portsentry
  * first stop the service with
	- sudo systemctl stop portsentry.service
  * then add exceptions to not to block various IP addresses ( at least you own)
	- edit file /etc/portsentry/portsentry.ignore.static
  * to use portsentry in advanced mode for the TCP and UDP, modify file /etc/default/portsentry:
	- TPC_MODE="atcp"
	- UDP_MODE="audp"
  * if someone is scanning your ports and u want ip to be banned: edit file /etc/portsentry/portsentry.conf
	- activate blocking by passing BLOCK_UDP and BLOCK_TCP to 1
  * ADD PORTSCAN JAIL + file /etc/fail2ban/filter.d/portscan.conf!
  * now start the service : sudo systemctl start portsentry.service
	
Services you need for this project: (stop everything else)
  * sudo systemctl list-unit-files --type=service --state=enabled
  	- apache2.service                        enabled //for web server
	- apparmor.service                       enabled //mandatory access controls
	- autovt@.service                        enabled //for login
	- cron.service                           enabled //for cron
	- dbus-org.freedesktop.timesync1.service enabled //Network Time Synchronization
	- fail2ban.service                       enabled //for fail2ban
	- getty@.service                         enabled //login
	- networking.service                     enabled //raises or downs the network interfaces
	- rsyslog.service                        enabled //for logs
	- ssh.service                            enabled //for ssh
	- sshd.service                           enabled //for ssh
	- syslog.service                         enabled //for logs
	- systemd-fsck-root.service              enabled-runtime //for file system checks
	- systemd-timesyncd.service              enabled //for synchronizing the system clock across the network
	- ufw.service                            enabled //for ufw

# Create the script that updates all packages:
  * create file update.sh and add scripts:
  	- sudo apt-get update -y >> /var/log/update_script.log
	- sudo apt-get upgrade -y >> /var/log/update_script.log
  * (You should first run update, then upgrade. Neither of them automatically runs the other.
	- apt-get update updates the list of available packages and their versions, but it does not install or upgrade any 		packages.
	- apt-get upgrade actually installs newer versions of the packages you have. After updating the lists, the package 		manager knows about available updates for the software you have installed. This is why you first want to 		update.)
	
# Schedule the script with crontab:
  * sudo crontab -e
  * add schelude :
  	- "0 4 * * 2 sudo /update.sh &" 
	- 0 for minutes, 4 for hours, 2 for weekday(wed) and the command, & is for running the command in background
	- "@reboot /update.sh &" ( executing the script when rebooting)
	
# Make a script that sends email if /etc/crontab file has been modified:
  * make file monitor_cron.sh
  	- make script that checks with diff if the crontab file is differet than crontab.bak file
	- diff abc.txt cba.txt  > /dev/null 2>&1 #not to print the out put of from diff command
	- if [ $? -eq 1 ]; then #system variable $? stores the return value of diff
		cat $file1 > $file2 #overrides the file2
		mail -s 'subject' root <<< 'message'
		fi
  * add one more crontab to run the script every midnight
  	- "0 0 * * * sudo /monitor_cron.sh
  * install mailutils and sendmail
  * Edit file /etc/aliases
	- root: root
	- run commans sudo newaliases
  
