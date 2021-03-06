

Problem statement 1:

1.

Find out the districts who achieved 100 percent objective in BPL cards

Export the results to mysql using sqoop  



Pig Script to calculate 100% Objective

--------------------------------------

File Name: StateObj.pig

-----------------------

REGISTER /usr/local/pig/lib/piggybank.jar;



DEFINE XPath org.apache.pig.piggybank.evaluation.xml.XPath();



Data =  LOAD 'statedev/state.xml' using org.apache.pig.piggybank.storage.XMLLoader('row') as (x:chararray);

StateDet = FOREACH Data GENERATE XPath(x, 'row/State_Name') AS statename, XPath(x, 'row/District_Name') AS disname,XPath(x, 'row/Project_Objectives_IHHL_BPL') AS BPL,XPath(x, 'row/Project_Objectives_IHHL_TOTAL') AS total ;

ObjFiltered = FILTER StateDet BY BPL == total;

STORE ObjFiltered INTO '/home/acadgild/StateObj' USING PigStorage(',');



Files Present after executing the script using : 

[acadgild@localhost ~]$pig -x local Stateobj.pig

[acadgild@localhost ~]$ ls /home/acadgild/StateObj

part-m-00000  _SUCCESS



Contents of output 

-------------------

[acadgild@localhost ~]$ cat /home/acadgild/StateObj/p*

Arunachal Pradesh,ANJAW,3232,3232

Arunachal Pradesh,DIBANG VALLEY,1085,1085

Arunachal Pradesh,KURUNG KUMEY,22036,22036

Arunachal Pradesh,LOHIT,8800,8800

Arunachal Pradesh,WEST SIANG,11472,11472

Bihar,BANKA,82439,82439

D & N Haveli,DADRA AND NAGAR HAVELI,2480,2480

Goa,NORTH GOA,15000,15000

Jammu & Kashmir,KARGIL,8475,8475

Jammu & Kashmir,KISHTWAR,22318,22318

Jammu & Kashmir,LEH (LADAKH),6090,6090

Jammu & Kashmir,REASI                                   ,21500,21500

Jammu & Kashmir,SAMBA                                   ,9849,9849

Jammu & Kashmir,SHOPIAN                                 ,10196,10196

[acadgild@localhost ~]$ 

[acadgild@localhost ~]$ 



Copy the output to HDFS

------------------------

[acadgild@localhost ~]$ hadoop fs -put StateObj /user/acadgild



Open another terminal:Terminal 2 and runl all the daemons using 

1) start-all.sh

2) Make sure the daemons are up using 

[acadgild@localhost ~]$ jps

16884 Jps

3382 SecondaryNameNode

3560 ResourceManager

3118 NameNode

3663 NodeManager

3215 DataNode



3) Run MySQL using the following commands

3.1) sudo service mysqld start

3.2) mysql -u root



Create a new Database:

mysql>create database state;

mysql>use state;



Create a table similar to the structure of the output file contents:

mysql>create table BPLObjectivesMet (State varchar(20), district varchar(50), BPL int, total int);



mysql>show tables;

+------------------+

| Tables_in_state  |

+------------------+

| BPLObjectivesMet |

| state80percent   |

+------------------+

2 rows in set (0.00 sec)



In Terminal1, run the sqoop command to Export the Pig Script analysis from HDFS to MySQL:

sqoop export --connect jdbc:mysql://localhost/state --username 'root' -P --table BPLObjectivesMet --export-dir '/user/acadgild/StateObj/part-m-00000' --input-fields-terminated-by ',' -m 1



In Terminal2, check for if the Tables have been populated with data:



mysql> select * from BPLObjectivesMet;

+-------------------+------------------------------------------+-------+-------+

| State             | district                                 | BPL   | total |

+-------------------+------------------------------------------+-------+-------+

| Arunachal Pradesh | ANJAW                                    |  3232 |  3232 |

| Arunachal Pradesh | DIBANG VALLEY                            |  1085 |  1085 |

| Arunachal Pradesh | KURUNG KUMEY                             | 22036 | 22036 |

| Arunachal Pradesh | LOHIT                                    |  8800 |  8800 |

| Arunachal Pradesh | WEST SIANG                               | 11472 | 11472 |

| Bihar             | BANKA                                    | 82439 | 82439 |

| D & N Haveli      | DADRA AND NAGAR HAVELI                   |  2480 |  2480 |

| Goa               | NORTH GOA                                | 15000 | 15000 |

| Jammu & Kashmir   | KARGIL                                   |  8475 |  8475 |

| Jammu & Kashmir   | KISHTWAR                                 | 22318 | 22318 |

| Jammu & Kashmir   | LEH (LADAKH)                             |  6090 |  6090 |

| Jammu & Kashmir   | REASI                                    | 21500 | 21500 |

| Jammu & Kashmir   | SAMBA                                    |  9849 |  9849 |

| Jammu & Kashmir   | SHOPIAN                                  | 10196 | 10196 |

+-------------------+------------------------------------------+-------+-------+

14 rows in set (0.05 sec)

