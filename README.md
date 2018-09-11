# Sandeep-LinuxServerConfig from Udacity

An Item Catalog website was developed as part of Course 3, we need to serve the same as part of linux virtual machine instance

You can visit http://13.126.209.146 for the website deployed.

## Instructions for SSH access to the instance
1. Download Private Key from amazon light sail website
2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	for example: mv ~/Downloads/LightsailDefaultPrivateKey-ap-south-1.pem ~/.ssh/udacity_key.rsa
3. Open your terminal and type in
	chmod 600 ~/.ssh/udacity_key.rsa
4. In your terminal, type in
	ssh -i ~/.ssh/udacity_key.rsa ubuntu@13.126.209.146
5. Development Environment Information

	Public IP Address

	13.126.209.146
	
	Private Key ( is not provided here. )

## Create a new user named grader
1. sudo su
2. sudo adduser grader
	it will ask for a password - provide it, i have provided -> grader
3. touch /etc/sudoers.d/grader
4. vim /etc/sudoers.d/grader, type in -> grader ALL=(ALL:ALL) ALL, save and quit

## Set ssh login using key
1. user -> ssh-keygen command to generate a pair of keys, then save the private key in ~/.ssh on local machine
2. copy and paste -> id_rsa.pub on your virtual machine - light sail instance 

	$ su grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys

	Issue the below commands to give permissions for the files created
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys

	
3. reload SSH using -> sudo service ssh restart
4. now you can use ssh to login with the new user you created

	ssh -i [privateKeyFile] grader@13.126.209.146

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
	## The first step to change ssh port, i did not do, since it was logging me out of the server everytime and i had to create a new lightsail instance everytime
1. Use -> sudo vim /etc/ssh/sshd_config and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using -> sudo service sshd restart

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	## The first step to change ssh port, i did not do, since it was logging me out of the server everytime and i had to create a new lightsail instance everytime

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
 
## Configure the local timezone to UTC
1. Configure the time zone -> sudo dpkg-reconfigure tzdata

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache -> sudo apt-get install apache2
2. Install mod_wsgi -> sudo apt-get install python-setuptools libapache2-mod-wsgi
3. Restart Apache -> sudo service apache2 restart

## Install and configure PostgreSQL
1. Install PostgreSQL -> sudo apt-get install postgresql
2. Check if no remote connections are allowed -> sudo vim /etc/postgresql/9.5/main/pg_hba.conf
	change peer to trust for postgres user
	and issue this command - sudo service postgresql restart
3. Login as user "postgres" -> sudo su - postgres
4. Get into postgreSQL shell -> psql
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;

5. Set a password for user catalog
	

	postgres=# ALTER ROLE catalog WITH PASSWORD 'sandeep';

6. Give user "catalog" permission to "catalog" application database
	

	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;

7. Quit postgreSQL -> postgres=# \q
8. Exit from user "postgres" 
	
	exit

 
## Install git, clone and setup your Catalog App project.
1. Install Git using -> sudo apt-get install git
2. Move to the /var/www directory 
3. Create the application directory -> sudo mkdir SandeepApp
4. Move inside this directory using -> cd SandeepApp
5. Clone the Catalog App to the virtual machine -> sudo git clone https://github.com/sahegde/UdacityItemCatalog.git
6. Rename the project's name -> sudo mv ./UdacityItemCatalog/ sandeepApp/
7. Rename project.py to __init__.py -> sudo mv project.py __init__.py
8. Edit database_setup.py, __init__.py and initialItems.py and change -> engine = create_engine('sqlite:///restaurantmenu.db') to -> engine = create_engine('postgresql://catalog:sandeep@localhost/catalog')
9. Install pip -> sudo apt-get install python-pip
10. Use pip to install dependencies -> sudo pip install followed by sudo pip install --upgrade pip
11. Install psycopg2 -> sudo apt-get -qqy install postgresql python-psycopg2
12. Create database schema -> sudo python database_setup.py

## Configure and Enable a New Virtual Host
1. Create SandeepApp.conf to edit: -> sudo vim /etc/apache2/sites-available/SandeepApp.conf
2. Add the following lines of code to the file to configure the virtual host. 
<VirtualHost *:80>
  ServerName 13.126.209.146
  ServerAdmin sandeephegde1990@gmail.com
  WSGIScriptAlias / /var/www/SandeepApp/sandeepapp.wsgi
  <Directory /var/www/SandeepApp/SandeepApp/>
	Order allow,deny
	Allow from all
  </Directory>
  Alias /static /var/www/SandeepApp/SandeepApp/static
  <Directory /var/www/SandeepApp/SandeepApp/static/>
	Order allow,deny
	Allow from all
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
3. Enable the virtual host with the following command -> sudo a2ensite SandeepApp

## Create the .wsgi File
1. Create the .wsgi File under /var/www/SandeepApp: 
	

	cd /var/www/SandeepApp
	sudo vim sandeepapp.wsgi 

2. Add the following lines of code to the sandeepapp.wsgi file:
	
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/SandeepApp/SandeepApp")
	from SandeepApp import app as application
	application.secret_key = 'super_secret_key'


## Restart Apache
1. Restart Apache -> sudo service apache2 restart
