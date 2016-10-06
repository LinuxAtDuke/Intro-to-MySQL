Introduction to MySQL
=====================

*Version 1, 2016-10-05*

**Instructor**

Andy Ingham (andy.ingham AT duke.edu)

**Table of Contents**

1. [Lab 0: Creating a Personal Linux VM](#lab0)
2. [Unit 1: Accessing an instance / User management](#unit1)
3. [Lab 1: Initial user lockdown](#lab1)
4. [Unit 2: Databases, schema](#unit2)
5. [Unit 3: Adding/modifying tables and indexes](#unit3)
6. [Lab 2/3: Working with databases and tables](#lab2/3)
7. [Unit 4: writing queries](#unit4)
8. [Unit 5: evaluating basic security and performance](#unit5)


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
  
  	_shell>>_ mysql -u root -p
	[initial pw = bitnami]
		
  	https://dev.mysql.com/doc/refman/5.5/en/default-privileges.html

	_mysql>>_ SELECT User, Host, Password FROM mysql.user;

	| User | Host      | Password                                  |
	|:-----|:----------|:------------------------------------------|
	| root | localhost | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	| root | linux     | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	| root | 127.0.0.1 | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	| root | ::1       | *3792637D0995C22FC1AEF939DA506C5011EF2856 |
	|      | localhost |                                           |
	|      | linux     |                                           |

	
  * general structure of the DBMS
  
	_mysql>>_ status

	_mysql>>_ show status;

	_mysql>>_ show databases;

	_mysql>>_ use *DATABASE*;
		e.g. use test;

	_mysql>>_ show tables;
	  	
	*TAB COMPLETION*
	
	*COMMAND HISTORY*


<a name='lab1'></a>
## Lab 1 - Initial user lockdown

  * Login to MySQL as 'root', change that user's password, and remove unnecessary authorizations
	
	_shell>>_ mysql -u root -p
	
	_mysql>>_ SET PASSWORD FOR 'root'@'localhost' = PASSWORD('XXXXXXXXX');

	_mysql>>_ SELECT User, Host, Password FROM mysql.user;
		
	_mysql>>_ DELETE FROM mysql.user WHERE User !='root' OR Host !='localhost';

	_mysql>>_ SELECT User, Host, Password FROM mysql.user;


<a name='unit2'></a>
## Unit 2: Databases, schema
  * Removing, creating databases is very simple
  
	_mysql>>_ DROP DATABASE test;
	
	_mysql>>_ CREATE DATABASE COLAB_CLASS;

	_mysql>>_ show databases;
	
  * Schema development is best done via an ER diagram and a whiteboard - consider these:
	- what are the data elements? (tables)
	- what relationships do they have with one another?
	- what are the important attributes of the data elements? (fields in tables)
	- what are the data types and metadata (NULL allowed? defaults?) for the attributes
	- what will govern uniqueness in each table? (simple or compound primary keys?)
	- what queries are users going to run?
	- what indexes are needed (beyond those for the primary keys)?
	
  * Fine-tuning of schema...
	- referential integrity - data types consistent across linking fields (foreign keys)
	- data types should be as prescriptive and compact as possible
	- index creation should be done where needed, but not elsewhere
	- index creation always faster BEFORE data is loaded into the table
	- verify that data is reasonably normalized (e.g., data generally de-duplicated)

  * Some examples
  
	_mysql>>_ describe LCL_genotypes;

	| Field    | Type         | Null | Key | Default | Extra |
	|:---------|:-------------|:-----|:----|:--------|:------|
	| IID      | varchar(16)  | NO   | PRI | NULL    |       |
	| SNPpos   | varchar(512) | NO   | PRI | NULL    |       |
	| rsID     | varchar(256) | NO   | MUL | NULL    |       |
	| genotype | varchar(512) | NO   |     | NULL    |       |

	_mysql>>_ describe phenotypes;

	| Field             | Type           | Null | Key | Default | Extra |
	|:------------------|:---------------|:-----|:----|:--------|:------|
	| LCL\_ID            | varchar(16)    | NO   | PRI | NULL    |       |
	| phenotype         | varchar(128)   | NO   | PRI | NULL    |       |
	| phenotypic\_value1 | decimal(20,10) | YES  |     | NULL    |       |
	| phenotypic\_value2 | decimal(20,10) | YES  |     | NULL    |       |
	| phenotypic\_value3 | decimal(20,10) | YES  |     | NULL    |       |
	| phenotypic_mean   | decimal(20,10) | YES  |     | NULL    |       |

	_mysql>>_ describe snp;

	| Field              | Type                | Null | Key | Default | Extra |
	|:-------------------|:--------------------|:-----|:----|:--------|:------|
	| rsID               | varchar(256)        | NO   | PRI | NULL    |       |
	| Chromosome         | tinyint(3) unsigned | NO   |     | NULL    |       |
	| Position           | int(10) unsigned    | NO   |     | NULL    |       |
	| Allele1            | varchar(128)        | NO   |     | NULL    |       |
	| Allele2            | varchar(128)        | NO   |     | NULL    |       |
	| DistanceToNearGene | varchar(32)         | NO   |     | NULL    |       |
	| Gene               | varchar(32)         | NO   |     | NULL    |       |
	| SNPtype            | varchar(64)         | NO   |     | NULL    |       |


<a name='unit3'></a>
## Unit 3: Adding/modifying tables and indexes

  * Looking at the syntax for creating the above tables...

	CREATE TABLE \`LCL\_genotypes\` \(
	\\`IID\` varchar(16) NOT NULL,
	\\`SNPpos\` varchar(512) NOT NULL,
	\\`rsID\` varchar(256) NOT NULL,
	\\`genotype\` varchar(512) NOT NULL,
	PRIMARY KEY (\`IID\`,\`SNPpos\`),
	KEY \`idx_rsID\` (\`rsID\`)
	) ENGINE=InnoDB DEFAULT CHARSET=latin1;

	CREATE TABLE `phenotypes` (
	`LCL_ID` varchar(16) NOT NULL,
	`phenotype` varchar(128) NOT NULL,
	`phenotypic_value1` decimal(20,10) DEFAULT NULL,
	`phenotypic_value2` decimal(20,10) DEFAULT NULL,
	`phenotypic_value3` decimal(20,10) DEFAULT NULL,
	`phenotypic_mean` decimal(20,10) DEFAULT NULL,
	PRIMARY KEY (`LCL_ID`,`phenotype`)
	) ENGINE=InnoDB DEFAULT CHARSET=latin1;

	CREATE TABLE `snp` (
	`rsID` varchar(256) NOT NULL,
	`Chromosome` bigint(20) unsigned NOT NULL,
	`Position` int(10) unsigned NOT NULL,
	`Allele1` varchar(1024) NOT NULL,
	`Allele2` varchar(1024) NOT NULL,
	`DistanceToNearGene` varchar(1024) NOT NULL,
	`Gene` varchar(256) NOT NULL,
	`SNPtype` varchar(64) NOT NULL,
	PRIMARY KEY (`rsID`)
	) ENGINE=InnoDB DEFAULT CHARSET=latin1;

  * A brief tangent to discuss backups! (via 'mysqldump')

	_shell>>_ mysqldump --no-data COLAB\_CLASS > [PATH\_TO\_OUTPUT]/COLAB\_WITHOUT\_DATA.sql	
	
<a name='lab2/3'></a>
## Lab 2/3: Working with databases and tables

	_mysql>>_ DROP DATABASE test;
	
	_mysql>>_ CREATE DATABASE COLAB_CLASS;

	_mysql>>_ show databases;
	
	_mysql>>_ USE COLAB_CLASS;
