Introduction to MySQL
=====================

*Version 1, 2016-09-27*

**Instructor**

Andy Ingham

**Table of Contents**

1. [Lab 0: Creating a Personal Linux VM](#lab0)
2. [Unit 1: Access control / User management](#unit1)
3. [Lab 1: Accessing an instance; initial user lockdown](#lab1)
4. [creating databases](#unit2)
5. [developing schema](#unit3)
6. [adding/altering tables](#unit4)
7. [writing queries](#unit5)
8. [evaluating basic security and performance](#unit6)

<a name='lab0'></a>
## Lab 0 - Creating a personal Linux VM

1. Using a web browser, go to *https://vm-manage.oit.duke.edu*
2. Login using your Duke NetId.
3. Select "New" at the top of the page
4. Fill out the forms to create a new VM for this class.
  * At step (3), select 'LAMP Stack' for the Server.
  * At step (5), select 'Request VM'

The vm-manage web page will tell you the name for your VM. The web site will also tell you the initial username and password. You should connect via ssh.

Example: `ssh bitnami@colab-sbx-89.oit.duke.edu` [Entering password when prompted]

<a name='unit1'></a>
## Unit 1: Access control / User management
  * how access is controlled
  * general structure of the DBMS
  * ways to stipulate database, user
  	* commandline
  	* .my.cnf







<a name='lab1'></a>
## Lab 1 - Accessing an instance; initial user lockdown
	https://dev.mysql.com/doc/refman/5.5/en/default-privileges.html
	shell>> mysql -u root -p
		[initial pw = bitnami]
	mysql>> SELECT User, Host, Password FROM mysql.user;
	+------+-----------+-------------------------------------------+
	| User | Host      | Password                                  |
	+------+-----------+-------------------------------------------+
	| root | localhost | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	| root | linux     | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	| root | 127.0.0.1 | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	| root | ::1       | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	|      | localhost |                                           |
	|      | linux     |                                           |
	+------+-----------+-------------------------------------------+
	
	mysql>> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('XXXXXXXXX');
		<and note the difference in the output of the above "SELECT" query>
		
	mysql>>exit
	
	shell>>nano .my.cnf
	
	[mysql]
	user = root
	password = XXXXXXXXX

	[mysqldump]
	user = root
	password = XXXXXXXXX