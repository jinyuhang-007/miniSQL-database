# miniSQL

miniSQL is a compact mini sql based database. It is purely implemented by C++. miniSQL implements basic functions such as creating and deleting tables in the database, querying, and index creation etc.

## Usage

### Data types

Three basic data types:

-  int

- float

- char(n), n<=255

### Open database

```mysql
create database db_name;    # create and use database
use db_name;          	    # use database
```

### Quit miniSQL

```mysql
quit;
```

### Run MiniSQL Script File

```c++
execfile file_name;
```

### Runtime Environment

System: windows10

IDE: visual studio 2017

### Table Creation and Deletion

A table can define up to 32 attributes, and each attribute can be specified as unique or not.

Iit supports the primary key definition of a single attribute.

To create a table:

```mysql
create table table_name (
  column_name type ,
  column_name type ,
  ...
  column_name type ,
  primary key ( column_name )
);
```


For example:

```mysql
create table student (
	sno char(8),
    sname char(16) unique,
    sage int,
    sgender char (1),
    primary key ( sno )
);
```


To delete a table:

```mysql
drop table table_name;
```

For example:

```mysql
drop table student;
```



### Index Creation and Deletion

The B+ tree index is automatically established for the main attribute of the table, and the B+ tree index can be specified by the user through the SQL statement for the attribute declared as unique. Therefore, all B+ tree indexes are single-attribute and single-valued.

To create an index：

```mysql
create index index_name on table_name ( column_name );
create index stunameidx on student ( sname );  
```

To delete an index：

```mysql
drop index index_name;
drop index stunameidx;            
```

### Search

You can query by specifying multiple conditions connected with and, and support equivalent query for three types; support interval query for int and float types, considering the actual usage, for char(n) type, range query is not supported. The syntax is as follows:

```mysql
select * from table_name;  
select * from table_name where condition;
# condition：column_name op value and column_name op value … and column_name op value
# op in {<>, <, >, <=, >=}
select * from student;
select * from student where sno = ‘88888888’;
select * from student where sage > 20 and sgender = ‘F’ and ……;
```

For example:

```mysql
select * from student;
select * from student where sno = ‘88888888’;
select * from student where sage > 20 and sgender = ‘F’ and ……;
```

### Insert

```mysql
insert into table_name values ( value_1 , value_2 , … , value_n );
```

For example:

```mysql
insert into student values (‘12345678’,’wy’,22,’M’); 
```

### Delete

```mysql
delete from table_name;
delete from table_name where condition;
```

For example:

```mysql
delete from student;
delete from student where sno = ‘88888888’ and age > 20 and ……;
```



## Design

### Overall design
- API
- Intepreter
- Buffer Manager
- Catalog Manager
- Record Manager
- Index Manager

![img](https://github.com/Liang-ZX/miniSQL/blob/master/clip_image002.jpg)

The main program initializes Buffer Manager objects, Catalog Manager objects, and Record Manager object instances; reads in user input, and calls the Interpreter module for interpretation.

The Interpreter module interprets the input, judges the accuracy of grammar and semantics, and calls the API module to provide the interface.

The API module calls Index Manager, Record Manager, Catalog Manager, and outputs the result of correct execution and error information.

The relationship between the four modules is shown in the figure.

### Interpreter

#### Introduction

The Interpreter receives and interprets the commands entered by the user, generates the internal data structure representation of the commands, checks the grammatical correctness and semantic correctness of the commands, and calls the functions provided by the API layer to execute the commands.

#### Implementation

The core of the Interpreter module is to perform semantic analysis and syntax checking, and complete functions including exiting the database and executing command files.
In terms of semantic analysis, refer to https://github.com/callMeName/MiniSql to implement the getWord function, which realizes the function of splitting the strings entered by the user and returning them keyword by keyword.

```c++
string Interpreter::getWord(string &s, int &pos)
```

getWord will only return word by word, and the three operators ",", "(" and ")", and automatically filter the remaining operators. All returned content will be parsed as a string, requiring interprete to perform type conversion.

In terms of grammar check, follow the MySQL grammar specification, check word by word, and require the primary key to be given when creating a table. In addition, for the input data, the Catalog Manager is uniformly called to obtain the type information of each attribute of the table to perform type checking. If the command is legal, call the API to execute the command, otherwise, output an error message.

The Interpreter doesn't implement the function of executing command files, but hands it over to the main program. The main loop of the program in main unifies the command files and input instructions into a string of statements, and calls the interpreter for analysis.

It is worth mentioning that the API of the interpreter uses polymorphic pointers to give it greater flexibility.

### API

#### Implementation

The API module is the core of the entire system, and its main function is to provide an interface for executing miniSQL statements for the Interpreter layer to call. The interface takes the commands generated by Interpreter layer interpretation as input, determines the execution rules based on the information provided by the Catalog Manager, calls the corresponding interfaces provided by the Record Manager, Index Manager, and Catalog Manager for execution, and outputs correct execution results or error messages.

### Catalog Manager

#### Introduction

Catalog Manager is responsible for managing all schema information for the database, including：

1. The definition information of all tables in the database, including the name of the table, the number of fields (columns) in the table, the primary key, and the indexes defined on the table.

2. The definition information of each field in the table, including field type, whether it is unique, etc.

3. The definition of all indexes in the database, including the table to which the index is built, and the field on which the index is built.

Catalog Manager must also provide an interface for accessing and manipulating the above information for use by Interpreter and API modules.

### Record Manager

#### Introduction

Record Manager is responsible for managing the data files of the data in the record table. The main function is to realize the creation and deletion of data files, the insertion, deletion and search operations of records, and provide corresponding interfaces to the outside world. The search operation recorded in it requires the ability to support searches without conditions and searches with one condition (including equal value search, unequal value search and interval search).

### Index Manager

#### Introduction

The main function of Index Manager is to create an index on the unique attribute of the table to improve the efficiency of data retrieval. The index file adopts B+ tree data structure. Index Manager defines three classes: Node class, BPT class and Index_Manager class. The Node class defines each node in the B+ tree, and provides corresponding functions for the BPT class to call. The BPT class is responsible for managing the entire B+ tree, and implements functions such as data insertion, deletion, and search by calling member functions in Node. The Index_Manager class manages all index files on a table, and stores three different types of B+ tree indexes through three map containers. Index_Manager provides interfaces for creating index files, deleting index files, and inserting, deleting, and querying data to the outside world.

#### BPT class

The BPT class manages the entire B+ tree index, and saves the entire B+ tree through a pointer pointing to the root node of the B+ tree. The BPT class provides four interfaces for the Index_Manager class, namely Insert_Key, Delete_Key, Delete_All and Search_Key. The functions of the four interfaces are: insert key, delete key, delete all keys, and retrieve the address information corresponding to the key.

#### Inteface

| Interface    | Function                 |
| ------------ | ------------------------ |
| Create_Index | create a index file             |
| Drop_Index   | delete a index file             |
| Drop_All     | delete all the index files on the table     |
| Clear_Index  | clear all index file data on the table |
| Search       | query data                 |
| Insert       | insert data                   |
| Delete       | delete data                     |

Each Index_Manager object manages all indexes on a table. When creating an object, the constructor accepts a parameter of type string representing the name of the table, and then the constructor will read all the index files that already exist on the table from the hard disk into memory to construct B+ Tree index, the destructor deletes the B+ tree in memory and writes all indexes to the hard disk.

In terms of interaction, RecordManager updates the data in the index file when the data in the table changes through the Insert and Delete interfaces. The API implements creating indexes, deleting indexes, and deleting tables by calling the Create_Index, Drop_Index, Drop_All, and Clear_Index interfaces. The function of uploading all indexes and clearing all data in the index file. In addition, IndexManager obtains the mode information on the table through the interface provided by CatalogManager, and realizes data exchange with the hard disk through BufferManager.

### Buffer Manager

#### Introduction

Buffer Manager is responsible for buffer management, the main functions are:

- Read the specified data to the buffer or write the data from the buffer to the file (**Page on Demand**, request paging)

- Implement the **LRU** algorithm, select the appropriate page to replace when the buffer is full

- Record the status of each page in the buffer, such as whether it has been modified, etc.（**dirty bit**）

- Provide the **pin** function of the buffer page, and lock the page of the buffer, and it is not allowed to replace it

In order to improve the efficiency of disk I/O operations, the unit of interaction between the buffer and the file system is a block, and the block size is set to 4KB.

#### Implementation

Buffer Manager first defines the metadata structure of files and blocks, namely the sqlFile and sqlBlcok structures. Save the dirty bit and pin related information in the sqlBlock structure, and save a pointer to the corresponding file meta structure at the same time. Different sqlBlocks of the same file are connected by linked list. sqlFile saves information such as the file name and the number of blocks in the cache, and at the same time saves a pointer pointing to the block header in the corresponding cache of the file. Different sqlFiles of the same buffer manager are also connected by a linked list.

Buffer Manager also maintains a sqlBlock* Pool[bnum] to store all allocated data block metadata. The method of array is adopted, the address space is continuous, and it has higher efficiency when implementing subsequent algorithms such as LRU.

The external interface takes block num as the unit, and implements functions including reading files, writing files, deleting files, locking, unlocking, and writing all blocks back to disk.

#### Key Algorithms

##### LRU

Both ReadFile and WriteFile will open the corresponding file, then try to find a block, and write the file content into the cache. Finding a block calls the private method

```c++
sqlBlock* getUsableBlock(const string db_name, sqlFile* fileInfo); //LRU here
```

This function will first search in the existing cache, if the block data already exists, it will return the corresponding sqlBlock*, otherwise check whether the number of allocated blocks has exceeded the upper limit allowed, if not, malloc a new block to store Data; if so, use the LRU algorithm to replace the least recently used block, write its content back to disk (dirty write), and then read in the data to be processed. At the same time, perform timestamp updates on other data blocks already in the cache to ensure consistency.

In addition, if the block has been locked, it is not allowed to be swapped out, except when the database is closed, it will be forcibly released, otherwise it must be manually unlocked before it can be swapped out.

#### Page On Demand

Only when the LRU algorithm is executed or the database is closed, will the content of the block be written back to the disk forcibly, otherwise all user operations, including readFile, writeFile, etc., will only be completed in the buffer and will not be written back to the disk (logical read and write ). The on-demand paging here is relatively simple, with a single block as the unit, and does not perform other functions that require hardware support.

#### Dirty Bit

For the blocks read into the buffer, when the write operation is performed, the dirty bit of the block will be set to 1. When exchanging blocks with the disk, only the dirty blocks will be written, otherwise the blocks will be directly replaced by the new data coverage. The implementation of this part is in the function writeBlocktoDisk().
