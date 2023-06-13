# Server configuration for MySQL service

In this repository I'll explain how to configure a VM Ubuntu server for deploying a MySQL service.

## System

For this server I chose [Ubuntu Server 22.04](https://ubuntu.com/download/server "Ubuntu Server 22.04 Jammy Jellyfish download page") as a server.

The system properties configured in VirtualBox are the following ones:

-  **CPU cores**: at least 2 (1 should be good but the more the merrier).
-  **RAM**: at least 2048 MB (2 GB), depending on your host machine RAM.
-  **HARD DRIVE**: 12 GB is perfectly fine (VirtualBox recommends 10 GB).

For the network interfaces, since my school's network design does not permit use the _Bridged Adapter_, I configured 2 interfaces:

-  One set to **_NAT_** (connection to outside network).
-  The other one set to **_Host-only Adapter_** to be able to connect the host device to the Virtual Machine.

To configure the secondary network interface is necessary to configure it. To do it, it's needed to access the _Host Network Manager_ by `Ctrl + H` keyboard key combination.

![Host Network Manager](Images/VB-HostNetworkManager.png)

This configuration is recommended even though the network allows to use the _Bridged Adapter_, because the _Host-only Adapter_ will be configured in the system with a static IP so it will not care about if the network changes. For the connectivity interface could be set if wanted to.

## Server Installation

The process is pretty straight forward, just need to configure correctly the network interfaces' IP. Here is the process in screenshots.

-  Set the preferred language:

![Preferred language](Images/Installation%200.png)

-  Skip the installer update (may not appear if the `.iso` file is the latest):

![Skip installer update](Images/Installation%201.png)

-  Set the preferred keyboard layout:

![HPreferred keyboard layot](Images/Installation%202.png)

-  Set installation type (just leave it as default):

![Installation type](Images/Installation%203.png)

-  Network configuration:

![Network configuration](Images/Installation%204.png)

As you can see, set a static IP for the _Host-only Adapter_ (will be the one without IP, the _NAT_/_Bridged Adapter_ will have an IP assigned by DHCP).

![Set Static IP](Images/Installation%205.png)

-  Configure disk encryption (I just leave this as default always)

![Disk encryption](Images/Installation%206%20skip%20proxy%20and%20mirror.png)

(I skipped the proxy and mirrors steps, just leave them as they are)

-  Storage configuration (configuration resume)

![Storage configuration](Images/Installation%207.png)

-  Set system name, username and password

![System Information, host name, user name, password](Images/Installation%208%20skip%20pro%20ssh%20and%20packages.png)

(Again I skipped some steps before installation is executed, if you want to install OpenSSH server or other packages in the installation process you'll need to do some extra config here).

Once all these steps are done, **installation is executed and you'll only need to wait until it's completed**.

## Software Installation and Configuration

First of all it's required to execute an update before starting to install any software

```bash
	$ sudo apt update
	# Not necessary but recommended
	$ sudo apt upgrade -y
```

Supposedly, this Ubuntu Server version comes alredy with Openss-Server. You can verify it executing `ssh` in the terminal and if it's not execute its installation.

```bash
	# Check if SSH is installed
	$ ssh
	# If SSH is not installed
	$ sudo apt install -y openssh-server
```

Once system is updated, it's time to install the MySQL Server. Execute the following commands:

```bash
	$ sudo apt install -y mysql-server
	# Not necessary but recommended
	$ sudo mysql_secure_installation
```

The last command sets a secure password for the root user in the database manager. The next step consists to create a user for a remote connection:

```bash
	# Access with the root user
	$ mysql -u root -p
	# Create the new user. This user will have access from anywhere (%)
	mysql> CREATE USER 'remote'@'%' IDENTIFIED BY 'passwd';
	# Grant privileges to user. This example will grant ALL privileges on a specific database
	mysql> GRANT ALL PRIVILEGES ON [database_name].* TO'remote'@'%';
	# Apply privileges
	mysql> FLUSH PRIVILEGES;
	# Exit MySQL console
	mysql> \q
```

Notice that privileges may change depending on the type of user.

The last step to allow remote connections is to disable the `bind-address` from configuration files. Edit the following file and comment both `bind-address` lines (instead of commenting them you can set th IP to `0.0.0.0` or `*`, but this is the way I've done it)

```bash
	# Access with the root user
	$ sudo nano /etc/mysql/mysql.conf.d/mysqld.conf

	# Inside the file, under [mysqld] you'll see the following lines uncommented, so comment them with '#'
	# Uncommented lines:
	bind-address        = 127.0.0.1
	mysqlx-bind-address	= 127.0.0.1
	# Commented lines
	#bind-address			= 127.0.0.1
	#mysqlx-bind-address	= 127.0.0.1
```

Save the file (`Ctrl + o` to save the buffer, `Ctrl + x` to exit the editor) and restart the service with the following command:

```bash
	# Restart MySQL service
	$ sudo systemctl restart mysql
```

Once it's restarted you can test the connection with a bash terminal or MySQL Workbench with the host machine.

## Server created

Click in the following image to download a pre-configured machine with all the steps mentioned in this repository.

<h3><a href="https://drive.google.com/drive/folders/1IN-KcWHaR5TRlqhcHpo3ZPFvwZqisYrZ?usp=sharing" target="_blank"><img src="https://upload.wikimedia.org/wikipedia/commons/6/6a/Google_Drive_text_logo_grey.png" style="display: block;
												 margin-left: auto;
												 margin-right: auto;"/></a></h3>
