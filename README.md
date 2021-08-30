Introduction to MySQL (aka "Working with mySQL")
=====================

*Version 11, 2021-09-09*

*https://github.com/LinuxAtDuke/Intro-to-MySQL/*

**Instructor**

Mary Clair Thompson

**Table of Contents**

1. [Lab 0: Creating a Personal Linux VM](#lab0)
2. [Unit 1: Access control / User management](#lab05)
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


1. Navigate to *https://vcm.duke.edu/* in a browser
2. Login using your Duke NetId
3. Select "Reserve a VM" (near the middle of the page)
4. On the next screen, select the "Lamp Stack" link from the list under "Linux Apps"
5. If you encounter a pop-up window about SSH keys (which displays if you do not have them set up for your netID), you may need to select the less secure option until you've done that step (which is outside the scope of this class).
6. After agreeing to the Terms of Use, the VCM web page will display the name of your VM along with available usernames. __You must first connect to the University VPN (if not "on campus"), THEN initiate an ssh session as the Admin User (vcm) -- do this via the "Terminal" app on your Mac or via "PuTTY" (available at https://www.chiark.greenend.org.uk/~sgtatham/putty/ ) on your Windows machine.__
		  
*Example (after establishing a University VPN session, if off campus):* 


	ssh <YOUR NETID>@vcm-<YOUR VCM ID>.vm.duke.edu

  * Answer "yes" to "Are you sure you want to continue connecting (yes/no)?" and enter your password when prompted.



<a name='lab05'></a>
## Lab 0.5: Access control / User management
 
  * Change your user to root so that you can install software


    shell>> sudo -i

  * Install MySql
	

    shell>> apt-get install mysql-server
  
  * Access MySql as root user (root has All The Permissions)


    shell>> mysql -u root

  * Now you're able to interact with MySql. You should see a mysql prompt:


	mysql>>

  * You were able to access MySql _without_ a password, which is less than ideal (we'll change that momentarily). For more information about default privileges, see https://dev.mysql.com/doc/refman/8.0/en/default-privileges.html

  * Run the command below to verify that there is currently no authentication required to access your database.
  
      

    mysql>> SELECT Host, User, plugin, authentication_string from mysql.user where User='root';

    | Host      | User | plugin      | authentication\_string |
    |:----------|:-----|:------------|:-----------------------|
    | localhost | root | auth_socket |                        |

	
  * Here are a few other commands you can run to get a feel for the general structure of the DBMS:
  

	mysql>> status

	mysql>> show status;

	mysql>> show databases;

  * Notice that there's a database named 'mysql'. You can switch focus to that database with:


    mysql>> use mysql;

  * Now you can easily see tables within mysql:


    mysql>> show tables;




<a name='lab1'></a>
## Lab 1 - Initial user lockdown

  * Let's change the root user's password and remove unnecessary authorizations:

  

    mysql>> update mysql.user set plugin='mysql_native_password' where user='root' and host='localhost';

    mysql>> flush privileges;

    mysql>> SET PASSWORD FOR 'root'@'localhost' = 'INSERT YOUR PASSWORD HERE';

  * Rerun the following command, and note that the output is different:


    mysql>> SELECT Host, User, plugin, authentication_string from mysql.user where User='root';
	
    mysql>> SELECT Host, User, plugin, authentication_string from mysql.user;

  * Let's exit mysql and confirm that the password has been updated.


	mysql>> quit

	shell>> mysql -u root

  * You should get an 'access denied' message. From now on, you'll need to input your password when you want to access the database. The '-p' flag indicates to mysql that you want to access the database with a password:


	shell>> mysql -u root -p

	shell>> Enter password: 




  

<a name='unit2'></a>
## Unit 2: Databases, schema
  * The following commands illustrate how to add and remove databases:

  
    mysql>> CREATE DATABASE colab_class;

	mysql>> show databases;

    mysql>> DROP DATABASE colab_class;

    mysql>> show databases;
	
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

  * You can use the 'describe' keyword to examine table schemas. The examples below won't work in your database as we haven't added tables yet, but they illustrate some of the preceding concepts.
  

    mysql>> describe LCL_genotypes;

    | Field    | Type         | Null | Key | Default | Extra |
    |:---------|:-------------|:-----|:----|:--------|:------|
    | IID      | varchar(16)  | NO   | PRI | NULL    |       |
    | SNPpos   | varchar(512) | NO   | PRI | NULL    |       |
    | rsID     | varchar(256) | NO   | MUL | NULL    |       |
    | genotype | varchar(512) | NO   |     | NULL    |       |

    mysql>> describe phenotypes;

    | Field             | Type           | Null | Key | Default | Extra |
    |:------------------|:---------------|:-----|:----|:--------|:------|
    | LCL\_ID            | varchar(16)    | NO   | PRI | NULL    |       |
    | phenotype         | varchar(128)   | NO   | PRI | NULL    |       |
    | phenotypic\_value1 | decimal(20,10) | YES  |     | NULL    |       |
    | phenotypic\_value2 | decimal(20,10) | YES  |     | NULL    |       |
    | phenotypic\_value3 | decimal(20,10) | YES  |     | NULL    |       |
    | phenotypic_mean   | decimal(20,10) | YES  |     | NULL    |       |

    mysql>> describe snp;

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
  

	mysql>> CREATE INDEX idx_rsID ON LCL_genotypes(rsID);

		Query OK, 358244487 rows affected (2 hours 33 min 15.53 sec)
		Records: 358244487  Deleted: 0  Skipped: 0  Warnings: 0
  
	mysql>> SHOW INDEX from LCL_genotypes;

		+---------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
		| Table         | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
		+---------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
		| LCL_genotypes |          0 | PRIMARY  |            1 | IID         | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
		| LCL_genotypes |          0 | PRIMARY  |            2 | SNPpos      | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
		| LCL_genotypes |          1 | idx_rsID |            1 | rsID        | A         |           2 |     NULL | NULL   |      | BTREE      |         |               |
		+---------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
		3 rows in set (0.00 sec)

  * A brief tangent to discuss backups! (via 'mysqldump')

	
	shell>> mysqldump -p --no-data colab\_class > COLAB\_WITHOUT\_DATA.sql	
	
<a name='lab2/3'></a>
## Lab 2/3: Working with databases and tables

  * Let's create a database and add some data!
  

	mysql>> CREATE DATABASE colab_class;
	mysql>> show databases;

  * Before we grab some data, we need to run one more command that will help us out later. This will allow us to load data from a local source near the end of the lab:


	mysql>> SET GLOBAL local_infile=1;
	

  * We need data to add to our database--grab the class files from the github repository
  
	
	mysql>> quit
	shell>> cd
	shell>> git clone https://github.com/LinuxAtDuke/Intro-to-MySQL.git

  * Load the file into your MySQL instance:
	

	shell>> mysql -u root -p colab_class < /root/Intro-to-MySQL/COLAB\_WITHOUT\_DATA.sql
	
  * Let's take a look at the results of the manual import. We'll dive straight into the colab_class database by including the db name at the end of our login string. We'll also include the --local-infile tag with value 1 to indicate to mysql that we are going to allow loading data from local files.

	
    shell>> mysql --local-infile=1 -u root -p colab_class
	shell>> Enter password:
    mysql>> show tables;
	
	+-----------------------+
	| Tables_in_colab_class |
	+-----------------------+
	| LCL_characteristics   |
	| LCL_genotypes         |
	| eQTL                  |
	| gwas_results          |
	| phenotypes            |
	| snp                   |
	+-----------------------+
	6 rows in set (0.00 sec)

  * Let's take a look at the schema for one of the tables:


    mysql>> DESCRIBE LCL_genotypes;

    +----------+--------------+------+-----+---------+-------+
	| Field    | Type         | Null | Key | Default | Extra |
	+----------+--------------+------+-----+---------+-------+
	| IID      | varchar(16)  | NO   | PRI | NULL    |       |
	| SNPpos   | varchar(512) | NO   | PRI | NULL    |       |
	| rsID     | varchar(256) | NO   | MUL | NULL    |       |
	| genotype | varchar(512) | NO   |     | NULL    |       |
	+----------+--------------+------+-----+---------+-------+
	4 rows in set (0.00 sec)


  * Notice that there's no data in the table yet:


	mysql>> SELECT * from LCL_genotypes;

	Empty set (0.00 sec)
  * Before we add data, let's practice editing the table schema. We'll change data types for two columns to allow them to hold larger strings.
	

    mysql>> ALTER TABLE LCL_genotypes MODIFY genotype VARCHAR(2048) NOT NULL;
	
    mysql>> ALTER TABLE LCL_genotypes MODIFY SNPpos VARCHAR(767) NOT NULL;
	
    mysql>> DESCRIBE LCL_genotypes;

	+----------+---------------+------+-----+---------+-------+
	| Field    | Type          | Null | Key | Default | Extra |
	+----------+---------------+------+-----+---------+-------+
	| IID      | varchar(16)   | NO   | PRI | NULL    |       |
	| SNPpos   | varchar(767)  | NO   | PRI | NULL    |       |
	| rsID     | varchar(256)  | NO   | MUL | NULL    |       |
	| genotype | varchar(2048) | NO   |     | NULL    |       |
	+----------+---------------+------+-----+---------+-------+
	4 rows in set (0.01 sec)
		
Notice the difference in the output above from earlier!
	
    mysql>> DESCRIBE gwas_results;

	+------------------+--------------+------+-----+---------+-------+
	| Field            | Type         | Null | Key | Default | Extra |
	+------------------+--------------+------+-----+---------+-------+
	| rsID             | varchar(256) | NO   | PRI | NULL    |       |
	| phenotype        | varchar(128) | NO   | PRI | NULL    |       |
	| study_population | varchar(8)   | NO   | PRI | NULL    |       |
	| EMPpvalue        | double       | YES  |     | NULL    |       |
	| beta             | double       | YES  |     | NULL    |       |
	+------------------+--------------+------+-----+---------+-------+
	5 rows in set (0.00 sec)
	
    mysql>> ALTER TABLE gwas_results MODIFY study_population VARCHAR(16) NOT NULL;
		
    mysql>> DESCRIBE gwas_results;

	+------------------+--------------+------+-----+---------+-------+
	| Field            | Type         | Null | Key | Default | Extra |
	+------------------+--------------+------+-----+---------+-------+
	| rsID             | varchar(256) | NO   | PRI | NULL    |       |
	| phenotype        | varchar(128) | NO   | PRI | NULL    |       |
	| study_population | varchar(16)  | NO   | PRI | NULL    |       |
	| EMPpvalue        | double       | YES  |     | NULL    |       |
	| beta             | double       | YES  |     | NULL    |       |
	+------------------+--------------+------+-----+---------+-------+
	5 rows in set (0.01 sec)
		
	
<a name='unit4'></a>
## Unit 4: Populating database with data

  * There are a number of ways we can get data into a table. 
    * We can add data record by record; the syntax below shows the format of this command, where:
      * tbl_name is the name of the table into which we wish to add data
      * column_1, ..., column_n are the names of the columns to which we want to add data
      * (value_11, value_12, ..., value_1n),...,(value_m1, value_m2, ..., value_mn) are the m records we wish to add to the table. The order of the values in each record must match the specified order of the columns.
      
    
	INSERT INTO tbl_name (column_1, column_2, ..., column_n) VALUES(value_11, value_12, ..., value_1n),...,(value_m1, value_m2, ..., value_mn);
	
  * Here are a couple of examples you can try:


	mysql>> INSERT INTO LCL_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02463','10:60523:T:G','rs112920234','TT');
		
	mysql>> INSERT INTO LCL_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02466','10:60523:T:G','rs112920234','TT'),('HG02563','10:60523:T:G','rs112920234','TT'),('HG02567','10:60523:T:G','rs112920234','00');
	

  * We can also add data in bulk (from a local file). The syntax follows, where:
    * tbl_name is the name of the table into which we wish to load data
    * '/path/to/filename' is the _full path_ to the name of the file where data is located
    * '\<column delimiter\>' indicates the character(s) separating fields in the data
    

	LOAD DATA LOCAL INFILE '/path/to/filename' INTO TABLE tbl_name FIELDS TERMINATED BY '<column delimiter>';

  * You can try running the following example yourself to load data:
    
  
	mysql>> LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
	mysql>> select * from snp;	
	

  * It's not uncommon to run across errors when trying to load data:
  
	
	mysql>> INSERT INTO LCL_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG024638392382903957','10:60523:T:G','rs112920234','TT');
	mysql>> ERROR 1406 (22001): Data too long for column 'IID' at row 1
		
  * This error indicates that the table schema is incompatible with this entry. Let's take a look:


	mysql> describe LCL_genotypes;
	+----------+---------------+------+-----+---------+-------+
	| Field    | Type          | Null | Key | Default | Extra |
	+----------+---------------+------+-----+---------+-------+
	| IID      | varchar(16)   | NO   | PRI | NULL    |       |
	| SNPpos   | varchar(767)  | NO   | PRI | NULL    |       |
	| rsID     | varchar(256)  | NO   | MUL | NULL    |       |
	| genotype | varchar(2048) | NO   |     | NULL    |       |
	+----------+---------------+------+-----+---------+-------+
	4 rows in set (0.01 sec)
		
  * Notice that the IID field only allows 16 characters, but the entry 'HG024638392382903957' is significantly longer than that! MySQL won't allow you to insert this value as-is. If you'd like, you can change the table schema to allow more characters in IID (similar to the syntax we saw above).
		
  * In addition to adding records, we can also change records that already exist (either one at a time or in groups). The syntax follows:

        
    UPDATE tbl_name SET col_name1=expr1, col_name2=expr2, ... WHERE where_condition;
        
  * You can try the following example yourself:


 	mysql> UPDATE snp SET Position='60524' WHERE rsID='rs112920234';
    Query OK, 1 row affected (0.01 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
	
    mysql> select rsID, Position from snp;                                          
    +-------------+----------+
	| rsID        | Position |
	+-------------+----------+
	| rs112920234 |    60524 |
	| rs147855157 |    61372 |
	| rs536439816 |    61386 |
	| rs536478188 |    60803 |
	| rs569167217 |    60684 |
	+-------------+----------+
	5 rows in set (0.01 sec)
	
  * We can also remove records (either one at a time or in bunches).

        
	mysql> select * from snp;
	+-------------+------------+----------+---------+---------+----------------------+------------+------------+
	| rsID        | Chromosome | Position | Allele1 | Allele2 | DistanceToNearGene   | Gene       | SNPtype    |
	+-------------+------------+----------+---------+---------+----------------------+------------+------------+
	| rs112920234 |         10 |    60524 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
	| rs147855157 |         10 |    61372 | CA      | C       | .                    | .          | .          |
	| rs536439816 |         10 |    61386 | A       | G       | dist=NONE;dist=31442 | NONE,TUBB8 | intergenic |
	| rs536478188 |         10 |    60803 | G       | T       | dist=NONE;dist=32025 | NONE,TUBB8 | intergenic |
	| rs569167217 |         10 |    60684 | C       | A       | dist=NONE;dist=32144 | NONE,TUBB8 | intergenic |
	+-------------+------------+----------+---------+---------+----------------------+------------+------------+
	5 rows in set (0.00 sec)
	
  * The syntax for dropping a record follows. I _strongly_ recommend that you supply a where clause when you wish to drop records from a table--otherwise the delete command will drop _all_ records in the table!

  
	DELETE FROM tbl_name WHERE where_condition;
 

  * Try running the following command:


    mysql>> DELETE FROM snp WHERE rsID='rs112920234';
	Query OK, 1 row affected (0.01 sec)

	mysql> select * from snp;                     
	+-------------+------------+----------+---------+---------+----------------------+------------+------------+
	| rsID        | Chromosome | Position | Allele1 | Allele2 | DistanceToNearGene   | Gene       | SNPtype    |
	+-------------+------------+----------+---------+---------+----------------------+------------+------------+
	| rs147855157 |         10 |    61372 | CA      | C       | .                    | .          | .          |
	| rs536439816 |         10 |    61386 | A       | G       | dist=NONE;dist=31442 | NONE,TUBB8 | intergenic |
	| rs536478188 |         10 |    60803 | G       | T       | dist=NONE;dist=32025 | NONE,TUBB8 | intergenic |
	| rs569167217 |         10 |    60684 | C       | A       | dist=NONE;dist=32144 | NONE,TUBB8 | intergenic |
	+-------------+------------+----------+---------+---------+----------------------+------------+------------+
	4 rows in set (0.00 sec)		


<a name='lab4'></a>
## Lab 4: Add more data to your database


  * Let's add data to three tables...
  

	mysql>> LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
	
	mysql>> LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/lcl_genotypes-data.infile' INTO TABLE LCL_genotypes FIELDS TERMINATED BY '\t';
		
	mysql>> LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/phenotypes-data.infile' INTO TABLE phenotypes FIELDS TERMINATED BY '\t';

<a name='unit5'></a>
## Unit 5: Writing queries to retrieve data

  * Below are some examples of simple queries you can try yourself:
	

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


  * Here's a slightly more complex query:
         
          
	mysql> select * from LCL_genotypes WHERE IID LIKE 'HG0246%'; 
	+------------------+--------------+-------------+----------+
	| IID              | SNPpos       | rsID        | genotype |
	+------------------+--------------+-------------+----------+
	| HG02463          | 10:60523:T:G | rs112920234 | TT       |
	| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       |
	| HG02466          | 10:60523:T:G | rs112920234 | TT       |
	+------------------+--------------+-------------+----------+
	3 rows in set (0.00 sec)

  * JOIN guidance:  https://stackoverflow.com/questions/6294778/mysql-quick-breakdown-of-the-types-of-joins
  
  * More JOIN guidance:  https://www.javatpoint.com/mysql-join
  
  * The default JOIN in MySQL is an INNER JOIN
  

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

  * You can write output from your queries directly to a file on your machine!
	

	mysql> SELECT * INTO OUTFILE '/var/lib/mysql-files/colab_class_result.txt' FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n' FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
	Query OK, 5 rows affected (0.00 sec)
	
	mysql> quit
	Bye
	root@vcm-XXXX:~$ cat /var/lib/mysql-files/colab_class_result.txt
	"HG02463","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
	"HG02463839238290","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
	"HG02466","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
	"HG02563","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
	"HG02567","10:60523:T:G","rs112920234","00","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
	

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
	
