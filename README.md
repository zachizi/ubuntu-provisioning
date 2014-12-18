ubuntu-provisioning
===================

My personal steps of instructions for provisioning a new Ubuntu server w/ big additions from Linode's guide.

1. Make sure the server is actually on (how do I so often forget this?).
2. `sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y dist-upgrade` update the system.
3. `echo "someHostName" > /etc/hostname && hostname -F /etc/hostname` set the hostname.
4. `echo "[server-ip] [fqdn] [hostname]" >> /etc/hosts` add to /etc/hosts
5. `dpkg-reconfigure tzdata` configure timezone.
6. `adduser [username] && usermod -aG sudo [username]` add a non-root user.
7. `update-alternatives --config editor` update default editor
8. `visudo` & append `[username] All=(ALL) NOPASSWD: ALL` if you don't want pass prompt for sudo commands on [username].
9. log out of server, scp up a public key, log back in as [username]
10. `mkdir .ssh && mv id_rsa.pub ~/.ssh/authorized_keys`; `id_rsa.pub` being the public key just uploaded.
11. `chown -R [username]:[username] ~/.ssh && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` set proper permissions.
12. `sudo vim /etc/ssh/sshd_config` then enable `PasswordAuthentication no` and `PermitRootLogin no`.
13. `sudo service ssh restart`
14. `sudo nano /etc/iptables.firewall.rules` and add the following + any additional needed firewall rules:

	```bash
	*filter
	 
	#  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
	-A INPUT -i lo -j ACCEPT
	-A INPUT -d 127.0.0.0/8 -j REJECT
	 
	#  Accept all established inbound connections
	-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	 
	#  Allow all outbound traffic - you can modify this to only allow certain traffic
	-A OUTPUT -j ACCEPT
	 
	#  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
	-A INPUT -p tcp --dport 80 -j ACCEPT
	-A INPUT -p tcp --dport 443 -j ACCEPT
	 
	#  Allow incoming MongoDB connections of default port
	-A INPUT -p tcp --dport 27017 -j ACCEPT
	 
	#  Allow SSH connections
	#
	#  The -dport number should be the same port number you set in sshd_config
	#
	-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
	 
	#  Allow ping
	-A INPUT -p icmp --icmp-type echo-request -j ACCEPT
	 
	#  Log iptables denied calls
	-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
	 
	#  Drop all other inbound - default deny unless explicitly allowed policy
	-A INPUT -j DROP
	-A FORWARD -j DROP
	 
	COMMIT
	```

15. `sudo iptables-restore < /etc/iptables.firewall.rules` add new rules to iptables.
16. `sudo vim /etc/network/if-pre-up.d/firewall` and add:

	```bash
	#!/bin/sh
	/sbin/iptables-restore < /etc/iptables.firewall.rules
	```

17. `sudo chmod +x /etc/network/if-pre-up.d/firewall` make file executable so as to readd firewall rules on system boot.
18. `sudo apt-get install fail2ban` to kick anyone out with too many failed logins.
19. `sudo apt-get install git build-essential` probably a pretty good idea to have these two.
20. `git clone https://github.com/zjr/dotfiles-lin.git ~/.dotfiles` and make symlinks for the necessitites (bash_profile, bashrc, inputrc).
21. `sh <(curl https://j.mp/spf13-vim3 -L)` install spf13-vim (vim 'distro' w/ some niceties).
22. `curl https://raw.githubusercontent.com/creationix/nvm/v0.20.0/install.sh | bash` install nvm
23. `source ~/.bash_profile && nvm install 0.10 && nvm alias default 0.10` install nvm, node, npm.
21. Now is a good time to install any system wide npm packages (e.g. pm2, bower, &c.).
22. `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10` install MongoDB Pt. 1;
23. `echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list` install MongoDB Pt. 2;
24. `sudo apt-get update && sudo apt-get install -y mongodb-org` finally actually install MongoDB.
25. `sudo vim /etc/mongod.conf` and comment out `bind_ip = 127.0.0.1` if outside connections needed.
26. Now is a good time to set up DBs & any necessary users or specific DB config.
27. I think that does it… next would be more specific things.  Maybe install some gems, pip.
