#Security considerations

Setup web server invovles a lot of considerations, please read the following to make sure your web server is secure at some level, at least not to make very obvious mistake.

[security your server please read firstly carefully](https://www.digitalocean.com/community/tutorials/7-security-measures-to-protect-your-servers)

[10 Best Practices To Secure and Harden Your Apache Web Server](https://geekflare.com/10-best-practices-to-secure-and-harden-your-apache-web-server/)

[Apache Web Server Hardening & Security Guide](https://geekflare.com/apache-web-server-hardening-security/)

[how-to-set-up-apache-web-server-on-centos-7](https://devops.profitbricks.com/tutorials/how-to-set-up-apache-web-server-on-centos-7/)

[lsof utility](http://www.catonmat.net/blog/unix-utilities-lsof/)


#Setup Centos7 server step guide

1. setup account of centos7:

	- 1.0 reference
          - [securing centos vps] (http://zcourts.com/2013/05/27/securing-a-linux-centos-vps-in-10-minutes/#sthash.8u4hdc6p.dpbs).
          - [My First Ten minutes on Linux server](http://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/)
          - adduser vs useradd
          	- it depends on what your want.
          	- adduser do more things than useradd.
          		- adduser prompt to set user name
          		- adduser will create home directory for you.
          		- adduser will create group default same as username.
          		- adduser will prompt you about setting passwd.
          
    - 1.1  add user ${userAccount} into sudoers.
    
         - 1.1.1 remote ssh into the system with root.
         - 1.1.2 adduser ${userAccount} to create user ${userAccount}
         - 1.1.3 passwd ${userAccount} to set passwd
         - 1.1.4 create sudo group by **groupadd sudo** not **addgroup**.
         - 1.1.5 assign ${userAccount} into sudo group by: **usermod -a -G sudo ${userAccount}**
         - 1.1.6 add privilege for sudo group by edit /etc/sudoer, and add line **%sudo  ALL=(ALL)    ALL**
   - 1.2  enalbe ${userAccount} public key login instead of passwd.
   		  - 1.2.1 ssh-key-gen to creat public/private key on your own local computer. (don't run it on your vps).
   		  - 1.2.2 ssh-copy-id ${userAccount}@your-server. it will copy your local public key into your remote server ~/.ssh/authorizedKeys and setup the file mode accordingly. 
   		  - 1.2.3 make sure you can login your server by issue: `ssh ${userAccount}@your-server` without need to input password of ${userAccount}.
   		  - 1.2.4 make sure ${userAccount} have the sudo priviledge by issue `sudo -i` and then input passwd, you will because as root if everything goes well.
   		  - 1.2.5 edit the /etc/ssh/sshd_config add enable the following. 
   		  `
   		  PubkeyAuthentication yes`
   		  `AuthorizedKeysFile      .ssh/authorized_keys`
   		  `PermitRootLogin no`  
2. install apache httpd.
	- 2.0 reference
		- [how to install httpd on centos7](http://www.liquidweb.com/kb/how-to-install-apache-on-centos-7/)
	- 2.1 install the httpd
		- 2.1.1 install httpd by issue `sudo yum install httpd`
		- 2.1.2 please ack yes when system ask for your confirm.
	- 2.2 test the httpd if it works well.
		- 2.2.1 disable the firwalld firstly to avoid the impact of firewalld by issue `systemctl stop firewalld`
		- 2.2.2 issue `wget localhost` command on your vps server, you will probably get the 403 forbidden.
		- 2.2.3 find the the httpd config file by issue `rpm -ql httpd | grep httpd.conf` and find the config file location /etc/httpd/conf/httpd.conf
		- 2.2.4 find out the error log of httpd by find line start with ErrorLog you will probably find "logs/error_log"
		- 2.2.5 then search the error_log folder by isse `rpm -ql httpd | grep logs` and you will find the logs position on `/etc/httpd/logs`
		- 2.2.6 find out the root cause of 403 forbidden by `vim /etc/httpd/logs/error_log`
		- 2.2.7 you will find the following line `AH01276: Cannot serve directory /var/www/html/: No matching DirectoryIndex (index.html) found, and server-generated directory index forbidden by Options directive`
		- 2.2.8 fix the error by simply put index.html inside the /var/www/html/index.html. Of Course you can also resolve by using the options directive.
		- 2.2.9 test httpd again make sure it work.
		- 2.2.10 recovery the firewall by `systemctl start firewalld`
		
3. install tomcat8
	- 3.0 reference 
		- [how install tomcat8 on centos7](https://hostpresto.com/community/tutorials/install-and-configure-tomcat-8-on-centos-7/)
		- [	
Install Tomcat 8 to configure JAVA Application Server](https://www.server-world.info/en/note?os=CentOS_7&p=tomcat8)
		- [Install Tomcat 8 on CentOS 7](https://panovski.me/install-tomcat-8-on-centos-7/)
	- 3.1 install jdk
		- 3.1.1 `sudo yum install openjdk` it will install jdk 1.8 by the time being.
	- 3.2 setup the tomcat user & group.
		- 3.2.1 create tomcat group `sudo groupadd tomcat`
		- 3.2.2 create tomcat user and assign to the tomcat group with home dir within /opt/tomcat and no login allowed. `sudo useradd -M -s /bin/nologin -g tomcat -d /opt/tomcat tomcat` it will not creat the user home dir, which means we can create user with a home dir not exist on the vps.

	- 3.3 install tomcat
		- 3.3.1 download the tomcat 8 from tomcat offical website into tmp dir. `wget http://mirrors.cnnic.cn/apache/tomcat/tomcat-8/v8.5.4/bin/apache-tomcat-8.5.4.tar.gz` and extract it with command `tar -xvf apache-tomcat-8.5.4.tar.gz`
		- 3.3.2 move the apache-tomcat-8.5.4 to /opt/tomcat
		- 3.3.3 change ownershop of the folder tomcat by issue `chown -R tomcat:tomcat tomcat`
	
	- 3.4 setup the tomcat service module.
		- 3.4.0 centos has two unit location: /etc/systemd/system and /usr/lib/systemd/system/. generally speaking the /etc/systemd/system has the link reference to the service module or unit inside the /usr/lib/systemd/system/ by command `systemctl enable xxx.yyy` which will start the service by system start up. And `systemctl disable xxx.yyy` which will delete the softlink, therefor the service will not get start by system startup.
	
		- 3.4.1 create tomcat8.service inside dir /usr/lib/systemd/system/
			
			```
				# Systemd unit file for tomcat
				[Unit]
				Description=Apache Tomcat Web Application Container
				After=syslog.target network.target

				[Service]
				Type=forking
				
				Environment=JAVA_HOME=/usr/lib/jvm/jre
				Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
				Environment=CATALINA_HOME=/opt/tomcat
				Environment=CATALINA_BASE=/opt/tomcat
				Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
				Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

				ExecStart=/opt/tomcat/bin/startup.sh
				ExecStop=/bin/kill -15 $MAINPID

				User=tomcat
				Group=tomcat

				[Install]
				WantedBy=multi-user.target
			
			```
		- 3.4.2 test tomcat8.service work or not
			- stop firwalld firstly to avoid its impact of 8080 port tracfic by run `systemctl stop firewalld`
			- acess the tomcat 8080 port by your web browser. if everything goes well you can see the tomcat home page.
			- for security consideration please recovery the firwalld by `systemctl start firewalld`
		- 3.4.3 [make tomcat more safe](https://www.owasp.org/index.php/Securing_tomcat).
			- make the /opt/tomcat belong to tomcat:tomcat
			- <del> **make /opt/tomcat/conf dir to 400 readonly, no write.**</del>
			- make /opt/tomcat/temp and read/write the temp directly but no execute.
			- make /opt/tomcat/logs 300 only write/execute.
		- 3.4.4 more safe martierals list
			- [step by step guid](https://www.mulesoft.com/tcat/tomcat-security)
			- [15 Ways To Secure Apache Tomcat 8](https://www.upguard.com/articles/15-ways-to-secure-apache-tomcat-8)
			
	- 3.5 allow port 80 to go through firewall
		- 3.5.1 add port 80 to firewall white list
		
			```
				firewall-cmd --zone=public --add-port=80/tcp --permanent  
				firewall-cmd --reload 
			```
4. connect apache with tomcat
	- 4.1 setup apache web server virtual servers.
	- 4.2 in order to make our web server for multiple purpose and support multiple application, we need to setup virtual host at time being.
	- 4.3 referenece material is [how-to-set-up-apache-virtual-hosts-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-centos-7)
	- 4.4 setup apache web server virtual hosts
		- 4.4.1 create two folders:  sites-available & sites-enabled. sites-availables include the config of our virtual host, site-enabled is just an folder , its files inside is just soft-link to the sites-available.
			
			```
				sudo mkdir /etc/httpd/sites-available
				sudo mkdir /etc/httpd/sites-enabled
			```
		- 4.4.2 enable the site-enabled virutalhosts by edit /etc/httpd/conf/httpd.conf
			
			```
				add the line below
				IncludeOptional sites-enabled/*.conf
			```
		- 4.4.3 setup your first vhost file. say want to setup ${userAccount} website here. Copy and past the following into a file named ${userAccount}.cn.web.conf into sites-available.
			
			```
				<VirtualHost *:80 >
					ServerName ${server_ip}
					ServerAlias ${server_ip}
	
					ProxyPass "/" "http://localhost:8080/" max=300
					ProxyPassReverse "/" "http://localhost:8080/"
				</VirtualHost>
			```
		
		- 4.4.4 setup soft link to sites-enables `sudo ln -s /etc/httpd/sites-available/${userAccount}.cn.web.conf /etc/httpd/sites-enabled/`
		- 4.4.5 enalbe the httpProxy for apache httpd
			
			```
			it is enabled by default
			```

5. install mysql

	- Install Step
		- download the mysql rpm
			wget http://repo.mysql.com/mysql-community-release-el7-7.noarch.rpm
		
		- Install the rpm (i short for install h-> hash)
		
			rpm -ivh mysql-community-release-el7-7.noarch.rpm
		
		- Update the yum by use `sudo yum update`
		
		- Install mysql-server
		
			sudo yum install mysql-server
		- Security mysql-server
		
			 sudo mysql_secure_installation
			 
			 make sure, no root login, no test database, no anonymous user.
		
	- Create DB & User & right grant
		
		- create database testDB
		- create user 'testuser'@'localhost' identified by 'password'
		- grant all on testdb.* to 'testuser' identified by 'password'
		
	- Root Rescue
		- kill the existing mysql instance stop the mysqld service
		- enter the sqld safe mode.
		- As you may seen from the name sqld_safe, yes it is safe mode, which means you can do what ever you want.
		- systemctl stop mysqld
		- start the safe mode mysqld_safe  (sudo mysqld_safe --skip-grant-tabls&)
		- update users password
		
			```
				use mysql;
				update user set PASSWORD=PASSWORD("newpass") Where USER='root';
				//flush the privileges;
				flush privileges;
				exit;
			```
	- Mysql Performance Tune (mysqltuner.pl)
	
		SomeTime we may run into sutation where mysql performance poorly  and we need to dig the problem, its where mysqltunner.pl come into play. it can easy help your figure out why the mysql performance is bad and its root cause.
		
		- Its a perl script and it need the root privileges of Mysql.
		- Get it by ` wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/mysqltuner.pl`
		- Simply run it by `perl ./mysqltuner.pl`.
		- Of course it will ask you for root password of mysql
			 
	- Reference
		- [Install MySql on Centos7](https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7)
6. learning material:

7. firewall:
	- 7.1 [firewalld opertaion](https://linuxconfig.org/how-to-open-http-port-80-on-redhat-7-linux-using-firewall-cmd)

	- 7.2 [firewalld learn](https://www.certdepot.net/rhel7-get-started-firewalld/)

	- 7.3 [firewalld rule for redhat](http://www.tecmint.com/firewalld-rules-for-centos-7/)
