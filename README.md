Introduction to MySQL (aka "Working with mySQL")
=====================

*Version 10, 2021-01-27*

*https://github.com/LinuxAtDuke/Intro-to-MySQL/*

**Instructor**

Andy Ingham (andy.ingham AT duke.edu)

**Table of Contents**

1. [Lab 0: Creating a Personal Linux VM](#lab0)
2. [Unit 1: Access control / User management](#unit1)
3. [Lab 1: Initial user lockdown](#lab1)
4. [Unit 2: Databases, schema](#unit2)
5. [Unit 3: Adding/modifying tables and indexes](#unit3)
6. [Lab 2/3: Working with databases and tables](#lab2/3)
7. [Unit 4: Populating database with data](#unit4)
8. [Lab 4: Adding data to your database](#lab4)
9. [Unit 5: Writing queries to retrieve data](#unit5)
10. [Lab 5: Practice with INSERT, UPDATE, DELETE, and SELECT (with JOIN!)](#lab5)
11. [Unit 6: Useful ancillary information](#unit6)

<a name='lab0'></a>
## Lab 0 - Creating a personal Linux VM

  * A brief tangent to discuss architecture == https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/client-server-architecture.pdf


1. Using a web browser, go to *https://vcm.duke.edu/*
2. Login using your Duke NetId.
3. Select "Reserve a VM" (near the middle of the page)
4. On the next screen, select the "Lamp Stack" link from the list under "Linux Apps"
5. If you encounter a pop-up window about SSH keys (which displays if you do not have them set up for your netID), you may need to select the less secure option until you've done that step (which is outside the scope of this class).
6. After agreeing to the Terms of Use, the VCM web page will display the name of your VM along with available usernames. __You must first connect to the University VPN (if not "on campus"), THEN initiate an ssh session as the Admin User (vcm) -- do this via the "Terminal" app on your Mac or via "PuTTY" (available at https://www.chiark.greenend.org.uk/~sgtatham/putty/ ) on your Windows machine.__
		  
*Example (after establishing a University VPN session, if off campus):* `ssh vcm@vcm-1473.vm.duke.edu` [Answering "yes" to "Are you sure you want to continue connecting (yes/no)?" and then entering the password behind "View Password" when prompted]



<a name='unit1'></a>
## Unit 1: Access control / User management

  * how access is controlled (https://dev.mysql.com/doc/refman/8.0/en/default-privileges.html )
  
  	_shell>>_ sudo -i

  	_shell>>_ mysql -u root *(_NO INITIAL PASSWORD EXISTS_)*

	\[<br/><br/>
	__NOTE that MySQL may not be installed.  If an error is encountered here, install mysql with:__
	
	_shell>>_ apt install -y mysql-server
	
	_shell>>_ mysql -u root *(_NO INITIAL PASSWORD EXISTS_)*
	<br/><br/>\]

	_mysql>>_ SELECT Host, User, plugin, authentication_string from mysql.user where User='root';

	| Host      | User | plugin      | authentication\_string |
	|:----------|:-----|:------------|:-----------------------|
	| localhost | root | auth_socket |                        |

	
  * general structure of the DBMS
  
	_mysql>>_ status

	_mysql>>_ show status;

	_mysql>>_ show databases;

	_mysql>>_ use *DATABASE*;
		(e.g. use mysql;)

	_mysql>>_ show tables;
	  	
	*TAB COMPLETION*
	
	*COMMAND HISTORY*



<a name='lab1'></a>
## Lab 1 - Initial user lockdown

  * Login to MySQL as 'root', change that user's password, and remove unnecessary authorizations
  
  	_shell>>_ sudo -i

  	_shell>>_ mysql -u root *(_NO INITIAL PASSWORD EXISTS_)*

	_mysql>>_ update mysql.user set plugin='mysql_native_password' where user='root' and host='localhost';

	_mysql>>_ flush privileges;

	_mysql>>_ SET PASSWORD FOR 'root'@'localhost' = '_SUPER\_GREAT\_PASSWORD\_HERE_';

	_mysql>>_ SELECT Host, User, plugin, authentication_string from mysql.user where User='root';
	
		[take note of how this output looks different than it did before]
	
	_mysql>>_ SELECT Host, User, plugin, authentication_string from mysql.user;


<a name='unit2'></a>
## Unit 2: Databases, schema
  * Removing or creating databases is very simple
  
	_mysql>>_ CREATE DATABASE colab_class;

	_mysql>>_ show databases;
	
	_mysql>>_ DROP DATABASE colab_class;
	
	_mysql>>_ show databases;
	
  * Schema development is best done via an ER diagram and/or a whiteboard - consider these:
	- what are the entities? _(the "things" or "concepts" that form the basis of our data)_
	- what relationships do they have with one another?
	- what are the important attributes of the entities?
	- what are the data types and metadata _(is NULL allowed? are there default values?)_ for those attributes?
	- what will determine uniqueness in each table? _(will the primary key be simple or compound?)_
	- what queries are users likely to run? _(this will inform index creation)_
	- what indexes are needed? _(to supplement the primary key)_	
	
 * Some (albeit simple and somewhat silly) examples:
	- https://www.edrawsoft.com/templates/pdf/pet-store-er-diagram.pdf
		- https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/pet-store-example-schemaOwner.pdf
		- https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/pet-store-example-schemaPet.pdf
		- https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/pet-store-example-schemaPetClinic.pdf
		- ESPECIALLY PROBLEMATIC: https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/pet-store-example-schemaTreatments.pdf
		- IN LIGHT OF THE ABOVE: https://dzone.com/articles/how-to-handle-a-many-to-many-relationship-in-datab
		- https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/pet-store-example-schemaPetStore.pdf
	- https://www.safaribooksonline.com/library/view/learning-mysql/0596008643/ch04s04.html

  * A tutorial to help with schema development:
  	- http://www.anchor.com.au/hosting/support/CreatingAQuickMySQLRelationalDatabase

  * Fine-tuning of schema...
	- referential integrity - data types consistent across linking fields (foreign keys)
	- data types (https://dev.mysql.com/doc/refman/8.0/en/data-types.html) should be as prescriptive and compact as possible
	- index creation should be done where needed, but not elsewhere
	- index creation is always faster BEFORE data is loaded into the table
	- verify that data is "reasonably" normalized (e.g., data generally de-duplicated)

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

		CREATE TABLE `LCL_genotypes` (
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
  
	_mysql>>_ CREATE INDEX idx_rsID ON LCL_genotypes(rsID);

		Query OK, 358244487 rows affected (2 hours 33 min 15.53 sec)
		Records: 358244487  Deleted: 0  Skipped: 0  Warnings: 0
  
	_mysql>>_ SHOW INDEX from LCL_genotypes;

		+---------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
		| Table         | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
		+---------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
		| LCL_genotypes |          0 | PRIMARY  |            1 | IID         | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
		| LCL_genotypes |          0 | PRIMARY  |            2 | SNPpos      | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
		| LCL_genotypes |          1 | idx_rsID |            1 | rsID        | A         |           2 |     NULL | NULL   |      | BTREE      |         |               |
		+---------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
		3 rows in set (0.00 sec)

  * A brief tangent to discuss backups! (via 'mysqldump')

	_shell>>_ mysqldump -p --no-data colab\_class > COLAB\_WITHOUT\_DATA.sql	
	
<a name='lab2/3'></a>
## Lab 2/3: Working with databases and tables

  * Create new database and populate it...
  
	_mysql>>_ CREATE DATABASE colab_class;
	
	_mysql>>_ show databases;
	
	_mysql>>_ exit

  * grab the class files from the github repository
  
	_shell>>_ git clone https://github.com/LinuxAtDuke/Intro-to-MySQL.git

  * load the file into your MySQL instance
	
	_shell>>_ mysql -u root -p colab\_class < /root/Intro-to-MySQL/COLAB\_WITHOUT\_DATA.sql
	
  * now check out the results of the import
	
	_shell>>_ mysql -u root -p colab_class;
	
	_mysql>>_ show tables;
	
	_mysql>>_ DESCRIBE LCL_genotypes;
	
  * now manually modify the table schema
	
	_mysql>>_ ALTER TABLE LCL_genotypes MODIFY genotype VARCHAR(2048) NOT NULL;
	
	_mysql>>_ ALTER TABLE LCL_genotypes MODIFY SNPpos VARCHAR(767) NOT NULL;
	
	_mysql>>_ DESCRIBE LCL_genotypes;
		
		[take note of how this output looks different than it did before]
	
	_mysql>>_ DESCRIBE gwas_results;
	
	_mysql>>_ ALTER TABLE gwas\_results MODIFY study\_population VARCHAR(16) NOT NULL;
		
	_mysql>>_ DESCRIBE gwas_results;
		
		[take note of how this output looks different than it did before]
	
<a name='unit4'></a>
## Unit 4: Populating database with data

  * Data can be added either record by record...
	* _mysql>>_ INSERT INTO tbl\_name () VALUES();
		* E.g., _mysql>>_ INSERT INTO LCL\_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02463','10:60523:T:G','rs112920234','TT');
		
	* _mysql>>_ INSERT INTO tbl\_name (a,b,c) VALUES(1,2,3),(4,5,6),(7,8,9);
		* E.g., _mysql>>_ INSERT INTO LCL_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02466','10:60523:T:G','rs112920234','TT'),('HG02563','10:60523:T:G','rs112920234','TT'),('HG02567','10:60523:T:G','rs112920234','00');
	
	* _mysql>>_ INSERT INTO tbl\_name SET col\_name=expr, col\_name=expr, ...
		* E.g., _mysql>>_ INSERT INTO phenotypes SET LCL\_ID='HG02461', phenotype='Cells\_ml\_after\_3\_days', phenotypic\_value1='878000', phenotypic\_value2='732000', phenotypic\_value3='805000', phenotypic_mean='805000';
	
	
  * Or in bulk (from an INFILE)
	* _mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
		
	
  * __WATCH OUT FOR WARNINGS!__ [_NOTE: As of MySQL version 5.7, THIS COMMAND RETURNS A FATAL ERROR AS OPPOSED TO A WARNING_] E.g., _mysql>>_ INSERT INTO LCL\_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG024638392382903957','10:60523:T:G','rs112920234','TT');

		Query OK, 1 row affected, 1 warning (0.00 sec)
		
		mysql> show warnings;
		+---------+------+------------------------------------------+
		| Level   | Code | Message                                  |
		+---------+------+------------------------------------------+
		| Warning | 1265 | Data truncated for column 'IID' at row 1 |
		+---------+------+------------------------------------------+
		1 row in set (0.00 sec)
		
		mysql> select * from LCL_genotypes;                       
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
		
  * Also possible (obviously) to change records that already exist (either one at a time or in bunches)...

		mysql> UPDATE tbl_name SET col_name=expr, col_name=expr, ... WHERE where_condition
		E.g., UPDATE LCL_genotypes SET IID='HG0246383' WHERE IID='HG02463839238290';
		Query OK, 1 row affected (0.00 sec)
		Rows matched: 1  Changed: 1  Warnings: 0
	
		mysql> select * from LCL_genotypes;                                          
		+-----------+--------------+-------------+----------+
		| IID       | SNPpos       | rsID        | genotype |
		+-----------+--------------+-------------+----------+
		| HG02463   | 10:60523:T:G | rs112920234 | TT       |
		| HG0246383 | 10:60523:T:G | rs112920234 | TT       |
		| HG02466   | 10:60523:T:G | rs112920234 | TT       |
		| HG02563   | 10:60523:T:G | rs112920234 | TT       |
		| HG02567   | 10:60523:T:G | rs112920234 | 00       |
		+-----------+--------------+-------------+----------+
		5 rows in set (0.00 sec)
	
  * Or to remove records (either one at a time or in bunches).  [First lets look at the table contents BEFOREHAND]

		mysql> select * from phenotypes;
		+---------+-----------------------+--------------------+--------------------+-------------------+--------------------+
		| LCL_ID  | phenotype             | phenotypic_value1  | phenotypic_value2  | phenotypic_value3 | phenotypic_mean    |
		+---------+-----------------------+--------------------+--------------------+-------------------+--------------------+
		| HG02461 | Cells_ml_after_3_days |  878000.0000000000 |  732000.0000000000 | 805000.0000000000 |  805000.0000000000 |
		| HG02462 | Cells_ml_after_3_days |  742000.0000000000 |  453000.0000000000 | 348000.0000000000 |  514333.3000000000 |
		| HG02463 | Cells_ml_after_3_days | 1200000.0000000000 | 1140000.0000000000 | 960000.0000000000 | 1100000.0000000000 |
		+---------+-----------------------+--------------------+--------------------+-------------------+--------------------+
		3 rows in set (0.00 sec)
	
  * Now remove...
	* _mysql>_> DELETE FROM tbl\_name WHERE where\_condition; __MAKE SURE YOU SUPPLY A WHERE CLAUSE UNLESS YOU WANT TO DELETE ALL ROWS!__
		* E.g., mysql> DELETE FROM phenotypes WHERE LCL_ID='HG02463';
		
		Query OK, 1 row affected (0.01 sec)

  * How does it look now?

		mysql> select * from phenotypes;                     
		+---------+-----------------------+-------------------+-------------------+-------------------+-------------------+
		| LCL_ID  | phenotype             | phenotypic_value1 | phenotypic_value2 | phenotypic_value3 | phenotypic_mean   |
		+---------+-----------------------+-------------------+-------------------+-------------------+-------------------+
		| HG02461 | Cells_ml_after_3_days | 878000.0000000000 | 732000.0000000000 | 805000.0000000000 | 805000.0000000000 |
		| HG02462 | Cells_ml_after_3_days | 742000.0000000000 | 453000.0000000000 | 348000.0000000000 | 514333.3000000000 |
		+---------+-----------------------+-------------------+-------------------+-------------------+-------------------+
		2 rows in set (0.00 sec)		


<a name='lab4'></a>
## Lab 4: Adding data to your database

  * First, make the necessary configuration change to allow this functionality:
  
  	_mysql>>_ set global local_infile=true;
  		
  	_mysql>>_ show global variables like 'local_infile';
  		
  		+---------------+-------+
		| Variable_name | Value |
		+---------------+-------+
		| local_infile  | ON    |
		+---------------+-------+
		1 row in set (0.00 sec)
		
  * Re-launch mysql client with ability enabled from the client:
  
	_mysql>>_ exit
  
  	_shell>>_ mysql --local\_infile=1 -u root -p colab_class;

  * Quickly add data to three tables...
  
	_mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
	
	_mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/lcl\_genotypes-data.infile' INTO TABLE LCL\_genotypes FIELDS TERMINATED BY '\t';
		
	_mysql>>_ show warnings;
	
	_mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/phenotypes-data.infile' INTO TABLE phenotypes FIELDS TERMINATED BY '\t';

<a name='unit5'></a>
## Unit 5: Writing queries to retrieve data

  * Simplest queries
	
		mysql> select * from LCL_genotypes;
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
	
		mysql> SELECT IID,rsID from LCL_genotypes WHERE genotype = 'TT';
		+------------------+-------------+
		| IID              | rsID        |
		+------------------+-------------+
		| HG02463          | rs112920234 |
		| HG02463839238290 | rs112920234 |
		| HG02466          | rs112920234 |
		| HG02563          | rs112920234 |
		+------------------+-------------+
		4 rows in set (0.00 sec)
	
		mysql> SELECT COUNT(*) from snp;
		+----------+
		| COUNT(*) |
		+----------+
		|        5 |
		+----------+
		1 row in set (0.04 sec)
	
		mysql> select * from snp;
		+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| rsID        | Chromosome | Position | Allele1 | Allele2 | DistanceToNearGene   | Gene       | SNPtype    |
		+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| rs147855157 |         10 |    61372 | CA      | C       | .                    | .          | .          |
		| rs536439816 |         10 |    61386 | A       | G       | dist=NONE;dist=31442 | NONE,TUBB8 | intergenic |
		| rs536478188 |         10 |    60803 | G       | T       | dist=NONE;dist=32025 | NONE,TUBB8 | intergenic |
		| rs569167217 |         10 |    60684 | C       | A       | dist=NONE;dist=32144 | NONE,TUBB8 | intergenic |
		+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		5 rows in set (0.00 sec)

  * Slightly more complex queries
                   
		mysql> select * from LCL_genotypes WHERE IID LIKE 'HG0246%'; 
		+------------------+--------------+-------------+----------+
		| IID              | SNPpos       | rsID        | genotype |
		+------------------+--------------+-------------+----------+
		| HG02463          | 10:60523:T:G | rs112920234 | TT       |
		| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       |
		| HG02466          | 10:60523:T:G | rs112920234 | TT       |
		+------------------+--------------+-------------+----------+
		3 rows in set (0.00 sec)

  * "JOIN" GUIDANCE:  https://stackoverflow.com/questions/6294778/mysql-quick-breakdown-of-the-types-of-joins
  
  * MORE "JOIN" GUIDANCE:  https://www.javatpoint.com/mysql-join
  
  * (The default "JOIN" in MySQL is an "INNER JOIN")
  
		mysql> SELECT * FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		+------------------+--------------+-------------+----------+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| IID              | SNPpos       | rsID        | genotype | rsID        | Chromosome | Position | Allele1 | Allele2 | DistanceToNearGene   | Gene       | SNPtype    |
		+------------------+--------------+-------------+----------+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| HG02463          | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02466          | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02563          | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02567          | 10:60523:T:G | rs112920234 | 00       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		+------------------+--------------+-------------+----------+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		5 rows in set (0.00 sec)
		
		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		+------------------+----------+------------+
		| IID              | Position | Gene       |
		+------------------+----------+------------+
		| HG02463          |    60523 | NONE,TUBB8 |
		| HG02463839238290 |    60523 | NONE,TUBB8 |
		| HG02466          |    60523 | NONE,TUBB8 |
		| HG02563          |    60523 | NONE,TUBB8 |
		| HG02567          |    60523 | NONE,TUBB8 |
		+------------------+----------+------------+
		5 rows in set (0.00 sec)

		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID where LCL_genotypes.rsID = 'rs536478188';
		Empty set (0.00 sec)

		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID where snp.rsID = 'rs536478188';
		Empty set (0.00 sec)
		
		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID where IID = 'HG02466';
		+---------+----------+------------+
		| IID     | Position | Gene       |
		+---------+----------+------------+
		| HG02466 |    60523 | NONE,TUBB8 |
		+---------+----------+------------+
		1 row in set (0.00 sec)

  * What if I want the output to go directly into a file instead of to the screen?
	
		mysql> SELECT * INTO OUTFILE '/var/lib/mysql-files/colab_class_result.txt' \
		         FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' \
		         LINES TERMINATED BY '\n' \
		         FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		Query OK, 5 rows affected (0.00 sec)

		mysql> SELECT IID,Position,Gene INTO OUTFILE '/var/lib/mysql-files/colab_class_result2.txt' \
		         FIELDS TERMINATED BY '\t' OPTIONALLY ENCLOSED BY '' ESCAPED BY '' \
		         LINES TERMINATED BY '\n' \
		         FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		Query OK, 5 rows affected (0.00 sec)
		
		mysql> exit
		Bye
		root@vcm-XXXX:~$ cat /var/lib/mysql-files/colab_class_result.txt
		"HG02463","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02463839238290","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02466","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02563","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02567","10:60523:T:G","rs112920234","00","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"

		root@vcm-XXXX:~$ cat /var/lib/mysql-files/colab_class_result2.txt
		HG02463	60523	NONE,TUBB8
		HG02463839238290	60523	NONE,TUBB8
		HG02466	60523	NONE,TUBB8
		HG02563	60523	NONE,TUBB8
		HG02567	60523	NONE,TUBB8
	

<a name='lab5'></a>
## Lab 5: Practice with INSERT, UPDATE, DELETE, and SELECT (with JOIN!)

  * Take some time to play around with queries we've talked about above...

<a name='unit6'></a>
## Unit 6: Useful ancillary information

  * please note that VCM VMs now default to powering down every morning at 06:00 am, so if you can't connect (starting tomorrow), the first thing to do is to login to https://vcm.duke.edu to verify that your VM is actually powered on.

  * sudo -- allows certain commands to be run with elevated privileges.  First, without:

		vcm@vcm-XXXX:~$ service mysql restart
		==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
		Authentication is required to restart 'mysql.service'.
		Authenticating as: root
		Password:
	
  * And now, with:

		vcm@vcm-XXXX:~$ sudo service mysql restart
		vcm@vcm-XXXX:~$ ps -aef | grep mysql 

  * To REBOOT the server itself: _note that this can also be done from the VCM webUI via "Power off" and then "Power on"_

		vcm@vcm-XXXX:~$ sudo shutdown -r now
		Connection to vcm-XXXX.vm.duke.edu closed by remote host.
		Connection to vcm-XXXX.vm.duke.edu closed
	
  * To change the configuration of the MySQL server, edit the "my.cnf" file __AND THEN RESTART THE mysql PROCESS!!__

		vcm@vcm-XXXX:~$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
			[making necessary edits to the file and saving them]
		vcm@vcm-XXXX:~$ sudo service mysql restart
	
  * To check the error log, use "cat" (or "more" or "less"...)

		vcm@vcm-XXXX:~$ sudo cat /var/log/mysql/error.log
	
