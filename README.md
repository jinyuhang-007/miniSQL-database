# miniSQL

miniSQL is a compact SQL-based database written in C++. It operates through the terminal.

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

API模块是整个系统的核心，其主要功能为提供执行MiniSQL语句的接口，供Interpreter层调用。该接口以Interpreter层解释生成的命令内部表示为输入，根据Catalog Manager提供的信息确定执行规则，并调用Record Manager、Index Manager和Catalog Manager提供的相应接口进行执行，并输出正确执行结果或者错误信息。

### Catalog Manager

#### Introduction

Catalog Manager负责管理数据库的所有模式信息，包括：

1. 数据库中所有表的定义信息，包括表的名称、表中字段（列）数、主键、定义在该表上的索引。

2. 表中每个字段的定义信息，包括字段类型、是否唯一等。

3. 数据库中所有索引的定义，包括所属表、索引建立在那个字段上等。

Catalog Manager还必需提供访问及操作上述信息的接口，供Interpreter和API模块使用。

### Record Manager

#### Introduction

Record Manager负责管理记录表中数据的数据文件。主要功能为实现数据文件的创建与删除、记录的插入、删除与查找操作，并对外提供相应的接口。其中记录的查找操作要求能够支持不带条件的查找和带一个条件的查找（包括等值查找、不等值查找和区间查找）。

### Index Manager

#### Introduction

IndexManager模块的主要作用是在表格的unique属性上建立索引，以提高数据检索的效率，索引文件采用B+树的数据结构。IndexManager模块定义了三个类，分别为：Node类、BPT类和Index_Manager类。其中Node类定义了B+树中的每个节点，并且提供了相应的函数供BPT类进行调用。BPT类负责管理整棵B+树，通过调用Node中的成员函数实现数据的插入、删除和搜索等功能。Index_Manager类管理一张表格上的所有索引文件，通过三个map容器存储三个不同类型的B+树索引。Index_Manager对外界提供创建索引文件、删除索引文件、以及数据的插入、删除、查询等接口。

#### B+树类BPT

BPT类管理整棵B+树索引，通过一个指向B+树根节点的指针保存整棵B+树，BPT类对Index_Manager类提供了四个接口，分别为Insert_Key、Delete_Key、Delete_All和Search_Key。四者的功能分别为：插入key、删除key、删除所有key、检索key所对应的地址信息。

#### Inteface

| Interface    | Function                 |
| ------------ | ------------------------ |
| Create_Index | 创建索引文件             |
| Drop_Index   | 删除索引文件             |
| Drop_All     | 删除表上所有索引文件     |
| Clear_Index  | 清空表上所有索引文件数据 |
| Search       | 查询数据                 |
| Insert       | 插入                     |
| Delete       | 删除                     |

每个Index_Manager对象管理一张表格上的所有索引，创建对象时构造函数接受一个string类型的参数代表表格的名字，随后构造函数会将该表上已经存在的所有索引文件从硬盘读入内存构造B+树索引，析构函数将内存中的B+树删除并将所有索引写入硬盘。

在交互方面，RecordManager通过Insert和Delete两个接口在表格的数据发生变化的时候更新索引文件中的数据，API通过调用Create_Index、Drop_Index、Drop_All和Clear_Index四个接口分别实现创建索引、删除索引、删除表上所有索引、清空索引文件中所有数据的功能。另外，IndexManager通过CatalogManager提供的接口获得表上的模式信息，并通过BufferManager实现与硬盘之间的数据交换。

### BufferManager

#### Introduction

Buffer Manager负责缓冲区的管理，主要功能有：

1、根据需要，读取指定的数据到系统缓冲区或将缓冲区中的数据写出到文件（**Page on Demand**，请求式分页）

2、实现**LRU**算法，当缓冲区满时选择合适的页进行替换

3、记录缓冲区中各页的状态，如是否被修改过等（**dirty bit**）

4、提供缓冲区页的**pin**功能，及锁定缓冲区的页，不允许替换出去

为提高磁盘I/O操作的效率，缓冲区与文件系统交互的单位是块，块的大小设置为4KB。

#### Implementation

Buffer Manager模块首先定义了文件和块的元数据结构，即sqlFile和sqlBlcok结构。在sqlBlock结构中保存dirty bit和pin的相关信息，同时保存一个指向对应文件元结构的指针。同一个文件的不同sqlBlock采用链表方式连接。sqlFile保存文件名、在缓存中块数量等信息，同时保存指针指向该文件对应缓存中的块首，同一个buffer manager的不同sqlFile也采用链表方式连接。

Buffer Manager同时维护一个sqlBlock* Pool[bnum]用来储存所有分配的数据块元信息。采用数组的方式，地址空间连续，在后续LRU等算法实现时，有更高的效率。

对外接口以block num为单位，实现包括读文件、写文件、删除文件、加锁、解锁、将所有块写回磁盘等函数。

#### Key Algorithms

##### LRU

不论是ReadFile还是WriteFile都会打开相应文件，然后尝试找到一个块，把文件内容写入缓存中。寻找块会调用私有方法

```c++
sqlBlock* getUsableBlock(const string db_name, sqlFile* fileInfo); //LRU here
```

该函数会先在已有的缓存中查找，若已有该块数据则返回对应的sqlBlock*,否则检查已分配的块数量是否已经超过了允许的上限，若否，则重新malloc一个块来存放数据；若是，则使用LRU算法替换least recently used的块，将其内容写回磁盘（脏写），然后读入要处理的数据。同时对其它已在缓存区中的数据块执行时间戳更新，以保证一致性。

此外，如果块已经上锁，则不允许换出，除了在关闭数据库时，会强制释放，否则必须手动解锁后，才能换出。

#### Page On Demand请求式分页

只有执行LRU算法或者关闭数据库时，才会强制将块的内容写回磁盘中，否则用户的所有操作，包括readFile, writeFile等均只在缓冲区完成，而暂不写回磁盘中（逻辑读写）。这里的请求式分页较为简单，以单个块为单位，不执行其它需要硬件支持的功能。

#### Dirty Bit脏写

对于读入缓冲区中的块，在执行write操作时，会将块的脏位(dirty bit)置为1，与磁盘交换块时，只有脏的块才会写入，否则块直接被新的数据覆盖。这一部分的实现在函数writeBlocktoDisk()中。
