Introduction to MySQL
=====================

*Version 19, 2016-10-11*

**Instructor**

Andy Ingham (andy.ingham AT duke.edu)

**Table of Contents**

1. [Lab 0: Creating a Personal Linux VM](#lab0)
2. [Unit 1: Accessing an instance / User management](#unit1)
3. [Lab 1: Initial user lockdown](#lab1)
4. [Unit 2: Databases, schema](#unit2)
5. [Unit 3: Adding/modifying tables and indexes](#unit3)
6. [Lab 2/3: Working with databases and tables](#lab2/3)
7. [Unit 4: Populating database with data](#unit4)
8. [Lab 4: Adding data to our database](#lab4)
9. [Unit 5: writing queries](#unit5)
10. [Unit 6: evaluating basic security and performance](#unit6)


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

  * how access is controlled (https://dev.mysql.com/doc/refman/5.5/en/default-privileges.html )
  
  	_shell>>_ mysql -u root -p
  	_[initial pw = bitnami]_


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
	
	_mysql>>_ CREATE DATABASE colab_class;

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
  
	_mysql>>_ describe lcl_genotypes;

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

		CREATE TABLE `lcl_genotypes` (
		`IID` varchar(16) NOT NULL,
		`SNPpos` varchar(512) NOT NULL,
		`rsID` varchar(256) NOT NULL,
		`genotype` varchar(512) NOT NULL,
		PRIMARY KEY (`IID`,`SNPpos`),
		KEY `idx_rsID` (`rsID`)
		) ENGINE=InnoDB DEFAULT CHARSET=latin1;

		CREATE TABLE `phenotypes` (
		`lcl_ID` varchar(16) NOT NULL,
		`phenotype` varchar(128) NOT NULL,
		`phenotypic_value1` decimal(20,10) DEFAULT NULL,
		`phenotypic_value2` decimal(20,10) DEFAULT NULL,
		`phenotypic_value3` decimal(20,10) DEFAULT NULL,
		`phenotypic_mean` decimal(20,10) DEFAULT NULL,
		PRIMARY KEY (`lcl_ID`,`phenotype`)
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

  * How was the "idx_rsID" index actually created?
  
	_mysql>>_ CREATE INDEX idx_rsID ON lcl_genotypes(rsID);

		Query OK, 358244487 rows affected (2 hours 33 min 15.53 sec)
		Records: 358244487  Deleted: 0  Skipped: 0  Warnings: 0
		
	_mysql>>_ SHOW INDEX from lcl_genotypes;

  * A brief tangent to discuss backups! (via 'mysqldump')

	_shell>>_ mysqldump --no-data COLAB\_CLASS > COLAB\_WITHOUT\_DATA.sql	
	
<a name='lab2/3'></a>
## Lab 2/3: Working with databases and tables

  * Drop unneeded database, create our new one, and populate it...
  
	_mysql>>_ DROP DATABASE test;
	
	_mysql>>_ CREATE DATABASE colab_class;
	
	_mysql>>_ show databases;
	
	_mysql>>_ exit

  * grab the dump file (COLAB\_WITHOUT\_DATA.sql) from https://github.com/LinuxAtDuke/Intro-to-MySQL

  * upload the dump file to your VM.  __FOR EXAMPLE:__

		shell>> cd Downloads/Intro-to-MySQL-master/
		shell>> sftp bitnami@colab-sbx-29.oit.duke.edu
			bitnami@colab-sbx-29.oit.duke.edu's password: 
			Connected to colab-sbx-29.oit.duke.edu.
		sftp> put COLAB_WITHOUT_DATA.sql
			Uploading COLAB_WITHOUT_DATA.sql to /home/bitnami/COLAB_WITHOUT_DATA.sql
			COLAB_WITHOUT_DATA.sql                       100% 4717     4.6KB/s   00:00    
		sftp> exit

  * load the file into your MySQL instance
	
	_shell>>_ mysql -u root -p colab_class < COLAB\_WITHOUT\_DATA.sql
	
  * now check out the results of the import
	
	_shell>>_ mysql -u root -p colab_class;
	
	_mysql>>_ show tables;
	
	_mysql>>_ DESCRIBE lcl_genotypes;
	
  * now manually modify the table schema
	
	_mysql>>_ ALTER TABLE lcl_genotypes MODIFY genotype VARCHAR(2048) NOT NULL;
	
	_mysql>>_ ALTER TABLE lcl_genotypes MODIFY SNPpos VARCHAR(767) NOT NULL;
	
	_mysql>>_ DESCRIBE lcl_genotypes;
	
	_mysql>>_ DESCRIBE gwas_results;
	
	_mysql>>_ ALTER TABLE gwas\_results MODIFY study\_population VARCHAR(16) NOT NULL;
		
	_mysql>>_ DESCRIBE gwas_results;
	
<a name='unit4'></a>
## Unit 4: Populating database with data

  * Data can be added either record by record...
	* _mysql>>_ INSERT INTO tbl\_name () VALUES();
		* E.g, _mysql>>_ INSERT INTO lcl\_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02463','10:60523:T:G','rs112920234','TT');
		
	* _mysql>>_ INSERT INTO tbl\_name (a,b,c) VALUES(1,2,3),(4,5,6),(7,8,9);
		* E.g, _mysql>>_ INSERT INTO lcl_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02466','10:60523:T:G','rs112920234','TT'),('HG02563','10:60523:T:G','rs112920234','TT'),('HG02567','10:60523:T:G','rs112920234','00');
	
	* _mysql>>_ INSERT INTO tbl\_name SET col\_name=expr, col\_name=expr, ...
		* E.g, _mysql>>_ INSERT INTO phenotypes SET LCL\_ID='HG02461', phenotype='Cells\_ml\_after\_3\_days', phenotypic\_value1='878000', phenotypic\_value2='732000', phenotypic\_value3='805000', phenotypic_mean='805000';
	
	
  * Or in bulk (from an INFILE)
	* _mysql>>_ LOAD DATA LOCAL INFILE '/home/bitnami/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
		
	
  * __WATCH OUT FOR WARNINGS!__ E.g, _mysql>>_ INSERT INTO lcl\_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG024638392382903957','10:60523:T:G','rs112920234','TT');

		Query OK, 1 row affected, 1 warning (0.00 sec)
		
		mysql> show warnings;
		+---------+------+------------------------------------------+
		| Level   | Code | Message                                  |
		+---------+------+------------------------------------------+
		| Warning | 1265 | Data truncated for column 'IID' at row 1 |
		+---------+------+------------------------------------------+
		1 row in set (0.00 sec)

		mysql> select * from lcl_genotypes;                                                                                           
		+------------------+--------------+-------------+----------+
		| IID              | SNPpos       | rsID        | genotype |
		+------------------+--------------+-------------+----------+
		| HG02463          | 10:60523:T:G | rs112920234 | TT       |
		| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       |
		| HG02466          | 10:60523:T:G | rs112920234 | TT       |
		| HG02563          | 10:60523:T:G | rs112920234 | TT       |
		| HG02567          | 10:60523:T:G | rs112920234 | 00       |
		+------------------+--------------+-------------+----------+
		5 rows in set (0.00 sec)

<a name='lab4'></a>
## Lab 4: Adding data to our database

  * Quickly add data to three tables...
  
	_mysql>>_ LOAD DATA LOCAL INFILE '/home/bitnami/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';

	_mysql>>_LOAD DATA LOCAL INFILE '/home/bitnami/lcl\_genotypes-data.infile' INTO TABLE lcl\_genotypes FIELDS TERMINATED BY '\t';

	_mysql>>_LOAD DATA LOCAL INFILE '/home/bitnami/phenotypes-data.infile' INTO TABLE phenotypes FIELDS TERMINATED BY '\t';
	
