# orcale数据库实验六-成都大学信息系统



<!-- [TOC] -->

## 一. 概述

学生信息管理系统是学校管理的重要工具，是学校不可或缺的一部分。随着在校人数的不断增加，教务系统的数量也不断的上涨。学校工作繁杂，资料众多，人工管理信息的难度也越来越大，显然是不能满足实际的需要，效率也是很低的。并且这种传统的方式存在着众多的弊端，如：保密性差.查询不便.效率低，很难维护和更新等，然而，本系统针对以上的缺点能够极大的提高学生信息管理的效率，也是科学化.正规化的管理，与世界接轨的重要条件。所以如何自动高效地管理信息是这些年来许多人所研究的。

随着这些年电脑计算机的速度质的提高，成本的下降，IT互联网大众趋势的发展。我们使用电脑的高效率才处理数据信息成为可能。学生学籍管理系统的出现，正是管理人员与信息数据，计算机的进入互动时代的体现。友好的人机交互模式，清晰简明的图形界面，高效安全的操作使得我们对成千上万的的信息的管理得心入手。通过这个系统，可以做到信息的规范处理，科学统计和快速的查询，从而减少管理方面的工作量。毋庸置疑，切实有效的把计算机管理引入学校教务管理中，对于促进学校管理制度，提高学校教学质量与办学水平有着显著意义。

## 二. 需求与功能分析 

学生信息管理系统，可用于学校等机构的学生信息管理，查询，更新与维护，使用方便，易用性强。该系统实现的大致功能；用户登陆。提供了学生学籍信息的查询，添加，修改，删除；学生成绩的录入，修改，删除，查询班级排名，修改密码等功能。管理员管理拥有最高的权限。允许添加教师信息和课程信息等。其提供了简单.方便的操作。

## 三. 数据库设计

## 1. 添加用户及权限管理

oracle中的表就是一张存储数据的表。表空间是逻辑上的划分。方便管理的。

数据表空间 (Tablespace) 

存放数据总是需要空间， Oracle把一个数据库按功能划分若干空间来保存数据。当然数据存放在磁盘最终是以文件形式，所以一盘一个数据表空间包含一个以上的物理文件
数据表。

在仓库，我们可能有多间房子，每个房子又有多个货架，每架又有多层。 我们在数据库中存放数据，最终是数据表的单元来存储与管理的。
数据文件。

以上几个概念都是逻辑上的， 而数据文件则是物理上的。就是说，数据文件是真正“看得着的东西”，它在磁盘上以一个真实的文件体现。

创建表空间：

~~~
格式: create tablespace 表间名 datafile '数据文件名' size 表空间大小
                create tablespace data_test datafile 'e:\oracle\oradata\test\data_1.dbf' size 2000M;
                create tablespace idx_test datafile 'e:\oracle\oradata\test\idx_1.dbf' size 2000M;
                (*数据文件名 包含全路径, 表空间大小 2000M 表是 2000兆) 
~~~

建好tablespace, 就可以建用户

~~~
格式: create user 用户名 identified by 密码 default tablespace 表空间表;
                create user study identified by study default tablespace data_test;
                (*我们创建一个用户名为 study,密码为 study, 缺少表空间为 data_test -这是在第二步建好的.)
                (*缺省表空间表示 用户study今后的数据如果没有专门指出，其数据就保存在 data_test中, 也就是保存在对应的物理文件 e:\oracle\oradata\test\data_1.dbf中)
~~~

创建用户并指定表空间

~~~
CREATE USER cici IDENTIFIED BY cici PROFILE DEFAULT DEFAULT TABLESPACE CICI ACCOUNT UNLOCK;
create user jykl identified by jykl default tablespace jykl_data temporary tablespace jykl_temp;
授权给新用户
GRANT connect, resource TO cici;
grant create session to cici;
~~~

授权给新用户

~~~
  grant connect,resource to study; 
    --表示把 connect,resource权限授予study用户
    grant dba to study;
    --表示把 dba权限授予给 study
~~~

创建数据表

在上面，我们已建好了用户 study 我们现在进入该用户 

sqlplusw study/study@test   然后就可以在用户study中创建数据表了

格式: create table 数据表名 

###  创建student角色，并创建student_lft用户，并且给用户分配角色空间

~~~sql
CREATE ROLE student;
GRANT connect,resource,CREATE VIEW TO student;
CREATE USER user_lft IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
ALTER USER user_lft QUOTA 50M ON users;
GRANT student TO user_lft;
exit
~~~

​	创建结果

<img src="images\1.png" style="zoom:50%;" />

## 2. 通过新创建的用户student_lft连接到 pdborcl 

<img src="images/2.png" style="zoom:50%;" />

## 3. 创建teacher角色并创建teacher_lft用户

~~~sql
CREATE ROLE teacher;
CREATE USER teacher_lft IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
GRANT teacher TO teacher_lft;
~~~

创建结果

<img src="images/3.png" style="zoom:50%;" />

## 4. 利用新创建的用户student_lft创建了五个表

~~~sql
  CREATE TABLE "STUDENT_LFT"."CLASS" 
   (	"ID" NUMBER, 
	"NAME" VARCHAR2(50 BYTE)
   ) SEGMENT CREATION DEFERRED 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  TABLESPACE "USERS" ;


  CREATE TABLE "STUDENT_LFT"."CURRICULUM" 
   (	"ID" NUMBER, 
	"SUBJECT_ID" NUMBER, 
	"CLASS_ID" NUMBER, 
	"TEACHER_ID" NUMBER, 
	"TIME" DATE
   ) SEGMENT CREATION DEFERRED 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  TABLESPACE "USERS" ;
  
  CREATE TABLE "STUDENT_LFT"."MYCLASS" 
   (	"ID" NUMBER, 
	"NAME" VARCHAR2(50 BYTE)
   ) SEGMENT CREATION DEFERRED 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  TABLESPACE "USERS" ;
  
  CREATE TABLE "STUDENT_LFT"."STUDENT" 
   (	"ID" NUMBER, 
	"NAME" VARCHAR2(50 BYTE), 
	"CLASS_ID" NUMBER, 
	"AGE" NUMBER(*,0), 
	"SEX" VARCHAR2(50 BYTE)
   ) SEGMENT CREATION DEFERRED 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  TABLESPACE "USERS" ;
  
  CREATE TABLE "STUDENT_LFT"."SUBJECT" 
   (	"ID" NUMBER, 
	"NAME" VARCHAR2(50 BYTE), 
	"CREDIT" NUMBER(*,0)
   ) SEGMENT CREATION DEFERRED 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  TABLESPACE "USERS" ;

  CREATE TABLE "STUDENT_LFT"."TEACHER" 
   (	"ID" NUMBER, 
	"NAME" VARCHAR2(50 BYTE), 
	"INFORMATION" VARCHAR2(50 BYTE)
   ) SEGMENT CREATION DEFERRED 
  PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  TABLESPACE "USERS" ;
~~~

学生信息表

|   属性   |   字段   |   注解   |
| :------: | :------: | :------: |
|   编号   |    id    | 学生编号 |
|   姓名   |   name   | 学生姓名 |
| 班级编号 | class_id | 班级编号 |
|   年龄   |   age    | 学生年龄 |
|   性别   |   sex    | 学生性别 |

课程信息表

| 属性 |  字段  |   注解   |
| :--: | :----: | :------: |
| 编号 |   id   | 课程编号 |
| 名称 |  name  | 课程姓名 |
| 学分 | credit | 课程学分 |

课表信息表

|   属性   |    字段    |   注解   |
| :------: | :--------: | :------: |
|   编号   |     id     | 课表编号 |
| 课程编号 | subject_id | 课程编号 |
| 班级编号 |  class_id  | 班级编号 |
| 教师编号 | teacher_id | 教师编号 |
|   时间   |    time    | 上课时间 |

班级信息表

| 属性 | 字段 |   注解   |
| :--: | :--: | :------: |
| 编号 |  id  | 班级编号 |
| 名称 | name | 班级名称 |

教师信息表

| 属性 |    字段     |     注解     |
| :--: | :---------: | :----------: |
| 编号 |     id      |   教师编号   |
| 名称 |    name     |   教师姓名   |
| 信息 | information | 教师信息介绍 |

## 6. 创建视图

- 视图(view)，也称虚表, 不占用物理空间，这个也是相对概念，因为视图本身的定义语句还是要存储在数据字典里的。视图只有逻辑定义。每次使用的时候,只是重新执行SQL。

- 视图是从一个或多个实际表中获得的，这些表的数据存放在数据库中。那些用于产生视图的表叫做该视图的基表。一个视图也可以从另一个视图中产生。

- 视图的定义存在数据库中，与此定义相关的数据并没有再存一份于数据库中。通过视图看到的数据存放在基表中。

- 视图看上去非常象数据库的物理表，对它的操作同任何其它的表一样。当通过视图修改数据时，实际上是在改变基表中的数据；相反地，基表数据的改变也会自动反映在由基表产生的视图中。由于逻辑上的原因，有些Oracle视图可以修改对应的基表，有些则不能（仅仅能查询）。

- 还有一种视图：物化视图（MATERIALIZED VIEW ），也称实体化视图，快照 （8i 以前的说法） ，它是含有数据的，占用存储空间。

~~~sql
 CREATE OR REPLACE FORCE EDITIONABLE VIEW "STUDENT_LFT"."MYVIEW_CURRICULUM" ("ID") AS 
  SELECT id FROM curriculum;
  
CREATE OR REPLACE FORCE EDITIONABLE VIEW "STUDENT_LFT"."MYVIEW_MYCLASS" ("ID") AS 
  SELECT name FROM myclass;
  
CREATE OR REPLACE FORCE EDITIONABLE VIEW "STUDENT_LFT"."MYVIEW_STUDENT" ("ID") AS 
  SELECT name FROM student;
  
  CREATE OR REPLACE FORCE EDITIONABLE VIEW "STUDENT_LFT"."MYVIEW_SUBJECT" ("ID") AS 
  SELECT name FROM subject;
  
  CREATE OR REPLACE FORCE EDITIONABLE VIEW "STUDENT_LFT"."MYVIEW_TEACHER" ("ID") AS 
  SELECT name FROM teacher;
~~~

## 7. 将五个表的视图的SELECT对象权限授予teacher用户 

~~~sql
GRANT SELECT ON myview_MYCLASS TO teacher_lft;
GRANT SELECT ON myview_SURRICULUM TO teacher_lft;
GRANT SELECT ON myview_STUDENT TO teacher_lft;
GRANT SELECT ON myview_SUBJECT TO teacher_lft;
GRANT SELECT ON myview_TEACHER TO teacher_lft;
~~~

## 8. 向数据库中写入数据共计50000多个数据

### 8.1 班级表

~~~sql
-- 班级表
DECLARE
    maxnumber   CONSTANT NUMBER := 100;
    i           NUMBER := 1;
BEGIN
    FOR i IN 1..maxnumber LOOP
        INSERT INTO myclass ( id,name ) VALUES (
            i,
            concat('class_',i)
        ); --CONCAT('test',i)是将test与i进行拼接
    END LOOP;

    dbms_output.put_line(' 成功录入数据！ ');
    COMMIT;
END;


~~~

<img src="images/4.png" style="zoom:50%;" />

### 8.2 教师表

~~~sql
--教师表
DECLARE
    maxnumber   CONSTANT NUMBER := 100;
    i           NUMBER := 1;
BEGIN
    FOR i IN 1..maxnumber LOOP
        INSERT INTO teacher ( id,name,information ) VALUES (
            i,
            concat('name1_',i),
            concat('information1_',i)
        ); --CONCAT('test',i)是将test与i进行拼接

    END LOOP;

    dbms_output.put_line(' 成功录入数据！ ');
    COMMIT;
END;




~~~

<img src="images/7.png" style="zoom:50%;" />

### 8.3 学科表

~~~sql
-- 学科表
DECLARE
    maxnumber   CONSTANT NUMBER := 100;
    i           NUMBER := 1;
BEGIN
    FOR i IN 1..maxnumber LOOP
        INSERT INTO subject ( id,name,credit ) VALUES (
            i,
            concat('subject_name_',i),
            4
        ); --CONCAT('test',i)是将test与i进行拼接

    END LOOP;

    dbms_output.put_line(' 成功录入数据！ ');
    COMMIT;
END;

~~~

<img src="images/6.png" style="zoom:50%;" />

### 8.4 课程表

~~~sql
--课程表
DECLARE
    maxnumber   CONSTANT NUMBER := 1000;
    i           NUMBER := 1;
    my_subject_id NUMBER := 1;
    my_class_id NUMBER := 1;
    my_teacher_id NUMBER := 1;
BEGIN
    FOR i IN 1..maxnumber LOOP
    if mod(i,10)=0 then my_class_id:=my_class_id+1;
    end if;
    if mod(i,10)=0 then my_subject_id:=my_subject_id+1;
    end if;
    if mod(i,10)=0 then my_teacher_id:=my_teacher_id+1;
    end if;
        INSERT INTO curriculum ( id,subject_id,class_id,teacher_id ) VALUES (
            i,
            my_subject_id,
            my_class_id,
            my_teacher_id
        ); --CONCAT('test',i)是将test与i进行拼接

    END LOOP;
    
    dbms_output.put_line(' 成功录入数据！ ');
    COMMIT;
END;


~~~

<img src="images/8.png" style="zoom:50%;" />

### 8.5 学生表

~~~sql
--学生表
DECLARE
    maxnumber   CONSTANT NUMBER := 50000;
    i           NUMBER := 1;
    my_class_id NUMBER := 1;
BEGIN
    FOR i IN 1..maxnumber LOOP
    if mod(i,500)=0 then my_class_id:=my_class_id+1;
    end if;
        INSERT INTO student ( id,name,class_id,age,sex ) VALUES (
            i,
            concat('student_name_',i),
            my_class_id,
            20,
            'man'
        ); --CONCAT('test',i)是将test与i进行拼接

    END LOOP;

    dbms_output.put_line(' 成功录入数据！ ');
    COMMIT;
END;
~~~

<img src="images/5.png" style="zoom:50%;" />

## 9. PL/SQL设计

 过程和函数由以下4部分： 

- 签名或头
- 关键字IS或AS
- 局部声明（可选）
- BEGIN和END之间的过程体（包括异常处理程序）

 简单示例： 

~~~sql
create or replace procedure show_line(ip_line_length in number, ip_separator in varchar2)
is 
actual_line varchar2(150);
begin
     insert into t_user(id,name,sex)values(ip_line_length,ip_separator,ip_line_length);
     for idx in 1..ip_line_length loop
         actual_line := actual_line||ip_separator;
     end loop;
     dbms_output.put_line(actual_line);
exception when others then
          dbms_output.put_line(SQLERRM);
end;

~~~

如下调用：

```sql
begin    show_line(50,'=');end;/
```

在SQLPLUS里面调用：

```sql
SQL> BEGIN2        show_line(50,'=');3    END;
```

 几点说明：

1、参数没有指定长度，当有实际数据传递进来的时候，参数的长度才被确定。

2、局部声明为：actual_line varchar2(150);

3、使用命令SQL> show errors在SQLPLUS里面查看错误。

### 9.1 存储过程：

#### 9.1.1 查询班级人数

~~~sql
create or replace PACKAGE MyPack IS

  PROCEDURE numberOfQueries(my_class_id NUMBER);
 END MyPack;
 /
 create or replace PACKAGE BODY MyPack IS
   PROCEDURE numberOfQueries(my_class_id NUMBER)
   AS
   lft integer; 
   BEGIN
     select id into lft
     from student
     where class_id=my_class_id;
   COMMIT;
   END numberOfQueries;
 END MyPack;

 set serveroutput on
 DECLARE
  my_class_id NUMBER;  
 BEGIN
  my_class_id := 1;
  MYPACK.numberOfQueries (my_class_id) ; 
 END;
~~~

<img src="images/9.png" alt="image-20191126201711814" style="zoom:50%;" />

#### 9.1.2. 查询学生分数


 ~~~sql
create or replace procedure p_contract_purchase_import(
  V_IN_SUBCOMPANYID    in VARCHAR2,
  V_IN_PURCONTRACTMONEY  in NUMBER,                       
  V_IN_PARTYBNAME     in VARCHAR2, 
  v_o_ret     out number        )
 
 as
 V_SUPPLIERID      INTEGER; 
 V_PARTYBACCOUNT     VARCHAR2(100);
 V_SQLERRM     VARCHAR2(4000);
 
 begin
 
  v_o_ret := 1;  
  V_SUPPLIERID := '';
  V_PARTYBACCOUNT := '';
 
  if(V_IN_PARTYBNAME is not null) then
   begin
    select t.SUPPLIERID,t.PARTYBACCOUNT,t.PARTYBBANK ,t.PARTYBNAME 
     into V_SUPPLIERID,V_PARTYBACCOUNT,V_PARTYBBANK,V_PARTYBNAME
     from T_SUPPLIER t where t.PARTYBNAME=trim(V_IN_PARTYBNAME) and t.SUBCOMPANYID=trim(V_IN_SUBCOMPANYID);
 
   exception
    when others then
     v_o_ret := 4 ; 
     V_PARTYBNAME := V_IN_PARTYBNAME;
 
     V_SQLERRM := SQLERRM;
     INSERT INTO T_LOG_DBERR
     (ERRTIME, ERRMODEL, ERRDESC)
     VALUES
      (SYSDATE,
       'PROCEDURES',
       'p_contract_purchase_import:ret=' || v_o_ret ||','||
     V_SQLERRM);
     COMMIT;
   end ;
  end if;
 
   end ; 
   commit;
   v_o_ret :=0 ;
  return;
 
 EXCEPTION
  WHEN OTHERS THEN
   ROLLBACK;
   V_SQLERRM := SQLERRM;
   INSERT INTO T_LOG_DBERR
    (ERRTIME, ERRMODEL, ERRDESC)
   VALUES
    (SYSDATE,
    'PROCEDURES',
    'p_contract_purchase_import:ret=' || v_o_ret ||','||
    V_SQLERRM);
   COMMIT;
 
 end p_contract_purchase_import;
 ~~~

### 9.2. 创建函数

#### 9.2.1. 查询所有科目总学分

~~~sql
create or replace PACKAGE MyPack IS
 
  FUNCTION Get_AllCredit(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  
 END MyPack;
 
 FUNCTION Get_AllCredit(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
  AS
   N NUMBER(20,2);
   BEGIN
    SELECT SUM(subject.credit) into N FROM subject;
    RETURN N;
   END;
 END MyPack;


~~~

 <img src="images/10.png" alt="img" style="zoom:50%;" />

## 10. 数据库备份

**ORACLE数据库备份与恢复详解**

Oracle的备份与恢复有三种标准的模式，大致分为两 大类，备份恢复(物理上的)以及导入导出(逻辑上的)，而备份恢复又可以根据数据库的工作模式分为非归档模式(Nonarchivelog-style) 和归档模式(Archivelog-style),通常，我们把非归档模式称为冷备份，而相应的把归档模式称为热备份。

### 热备份和冷备份优缺点

#### 热备份的优点是：

 1．可在表空间或数据文件级备份，备份时间短。 

 2．备份时数据库仍可使用。 

 3．可达到秒级恢复（恢复到某一时间点上）。 

 4．可对几乎所有数据库实体作恢复。 

 5．恢复是快速的，在大多数情况下在数据库仍工作时恢复。 

#### 热备份的不足是：

 1．不能出错，否则后果严重。 

 2．若热备份不成功，所得结果不可用于时间点的恢复。 

 3．因难维护，所以要特别仔细小心，不允许“以失败而告终”。 

#### 冷备份的优点是：

 1．是非常快速的备份方法（只需拷贝文件） 

 2．容易归档（简单拷贝即可） 

 3．容易恢复到某个时间点上（只需将文件再拷贝回去） 

 4．能与归档方法相结合，作数据库“最新状态”的恢复。 

 5．低度维护，高度安全。 

#### 冷备份不足是：

 1．单独使用时，只能提供到“某一时间点上”的恢复。 

 2．在实施备份的全过程中，数据库必须要作备份而不能作其它工作。也就是说，数据库必须是关闭状态。


 3．若磁盘空间有限，只能拷贝到磁带等其它外部存储设备上，速度会很慢。 

 4．不能按表或按用户恢复。

### 物理备份之冷备份：

   当数据库可以暂时处于关闭状态时，我们需要将它在这一稳定时刻的数据相关文件转移到安全的区域，当数据库遭到破坏，再从安全区域将备份的数据库相关文件拷 贝回原来的位置，这样，就完成了一次快捷安全等数据转移。由于是在数据库不提供服务的关闭状态，所以称为冷备份。冷备份具有很多优良特性，比如上面图中我 们提到的，快速，方便，以及高效。一次完整的冷备份步骤应该是：

   1，首先关闭数据库（shutdown normal）

   2，拷贝相关文件到安全区域（利用操作系统命令拷贝数据库的所有的数据文件、日志文件、控制文件、参数文件、口令文件等（包括路径））

   3，重新启动数据库（startup）

   以上的步骤我们可以用一个脚本来完成操作：

~~~
   su – oracle <   sqlplus /nolog 
   connect / as sysdba
   shutdown immediate;
   !cp 文件  备份位置（所有的日志、数据、控制及参数文件）;
   startup;
   exit;
~~~

   这样，我们就完成了一次冷备份，请确定你对这些相应的目录（包括写入的目标文件夹）有相应的权限。

   恢复的时候，相对比较简单了，我们停掉数据库，将文件拷贝回相应位置，重启数据库就可以了，当然也可以用脚本来完成。

### 10.1. 全数据库备份

~~~
[oracle@oracle-pc ~]$ cat rman_level0.sh
 [oracle@oracle-pc ~]$ ./rman_level0.sh
~~~

<img src="images/11.png" alt="img" style="zoom:50%;" />

<img src="images/12.png" alt="img" style="zoom:50%;" />

### 10.2. 查询备份文件

~~~
[oracle@oracle-pc ~]$ cd rman_backup
 
 c ~]$ cd rman_backup
 [oracle@oracle-pc rman_backup]$ ll
 总用量 777132
 -rw-rw---- 1 oracle oracle   5632 11月 26 15:17 arclv0_ORCL_20191126_9cuhrjre_1_1.bak
 -rw-rw---- 1 oracle oracle 18644992 11月 26 15:17 c-1392946895-20191126-00
 -rw-rw---- 1 oracle oracle 225976320 11月 26 15:15 dblv0_ORCL_20191126_99uhrjo4_1_1.bak
 -rw-rw---- 1 oracle oracle 386334720 11月 26 15:16 dblv0_ORCL_20191126_9auhrjp7_1_1.bak
 -rw-rw---- 1 oracle oracle 164716544 11月 26 15:16 dblv0_ORCL_20191126_9buhrjqk_1_1.bak
 -rw-rw-r-- 1 oracle oracle   51769 11月 11 00:36 lv0_20191111-003303_L0.log
 -rw-rw-r-- 1 oracle oracle   21867 11月 26 15:17 lv0_20191126-151506_L0.log
 -rw-rw-r-- 1 oracle oracle   7708 11月 11 00:37 lv1_20191111-003650_L0.log
~~~

<img src="images/13.png" alt="img" style="zoom:50%;" />

### 10.3. 查看备份文件的内容

~~~
[oracle@oracle-pc rman_backup]$ rman target /
 
 Recovery Manager: Release 12.1.0.2.0 - Production on 星期二 11月 26 15:39:01 2019
 
 Copyright (c) 1982, 2014, Oracle and/or its affiliates. All rights reserved.
 
 connected to target database: ORCL (DBID=1392946895)
 
 RMAN> list backup;
 
 using target database control file instead of recovery catalog
 
 List of Backup Sets
 ===================
 
 
 BS Key Type LV Size    Device Type Elapsed Time Completion Time
 ------- ---- -- ---------- ----------- ------------ ---------------
 249   Incr 0 215.50M  DISK    00:00:32   26-11月-19  
     BP Key: 249  Status: AVAILABLE Compressed: YES Tag: TAG20191126T151516
     Piece Name: /home/oracle/rman_backup/dblv0_ORCL_20191126_99uhrjo4_1_1.bak
  List of Datafiles in backup set 249
  Container ID: 3, PDB Name: PDBORCL
  File LV Type Ckp SCN  Ckp Time  Name
  ---- -- ---- ---------- ---------- ----
  8  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/system01.dbf
  9  0  Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/sysaux01.dbf
  10  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf
  11  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/example01.dbf
  12  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_1.dbf
  13  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_2.dbf
  16  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_3.dbf
  17  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_4.dbf
  77  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users03_1.dbf
  78  0 Incr 47696807  26-11月-19 /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users03_2.dbf
 
 BS Key Type LV Size    Device Type Elapsed Time Completion Time
 ------- ---- -- ---------- ----------- ------------ ---------------
 250   Incr 0 368.43M  DISK    00:00:43   26-11月-19  
     BP Key: 250  Status: AVAILABLE Compressed: YES Tag: TAG20191126T151516
     Piece Name: /home/oracle/rman_backup/dblv0_ORCL_20191126_9auhrjp7_1_1.bak
  List of Datafiles in backup set 250
  File LV Type Ckp SCN  Ckp Time  Name
  ---- -- ---- ---------- ---------- ----
  1  0 Incr 47696822  26-11月-19 /home/oracle/app/oracle/oradata/orcl/system01.dbf
  3  0 Incr 47696822  26-11月-19 /home/oracle/app/oracle/oradata/orcl/sysaux01.dbf
  4  0 Incr 47696822  26-11月-19 /home/oracle/app/oracle/oradata/orcl/undotbs01.dbf
  6  0 Incr 47696822  26-11月-19 /home/oracle/app/oracle/oradata/orcl/users01.dbf
 
 BS Key Type LV Size    Device Type Elapsed Time Completion Time
 ------- ---- -- ---------- ----------- ------------ ---------------
 251   Incr 0 157.08M  DISK    00:00:18   26-11月-19  
     BP Key: 251  Status: AVAILABLE Compressed: YES Tag: TAG20191126T151516
     Piece Name: /home/oracle/rman_backup/dblv0_ORCL_20191126_9buhrjqk_1_1.bak
  List of Datafiles in backup set 251
  Container ID: 2, PDB Name: PDB$SEED
  File LV Type Ckp SCN  Ckp Time  Name
  ---- -- ---- ---------- ---------- ----
  5  0 Incr 1819408  01-12月-14 /home/oracle/app/oracle/oradata/orcl/pdbseed/system01.dbf
  7  0 Incr 1819408  01-12月-14 /home/oracle/app/oracle/oradata/orcl/pdbseed/sysaux01.dbf
 
 BS Key Size    Device Type Elapsed Time Completion Time
 ------- ---------- ----------- ------------ ---------------
 252   5.00K   DISK    00:00:00   26-11月-19  
     BP Key: 252  Status: AVAILABLE Compressed: YES Tag: TAG20191126T151702
     Piece Name: /home/oracle/rman_backup/arclv0_ORCL_20191126_9cuhrjre_1_1.bak
 
  List of Archived Logs in backup set 252
  Thrd Seq   Low SCN  Low Time  Next SCN  Next Time
  ---- ------- ---------- ---------- ---------- ---------
  1  1614  47696796  26-11月-19 47696850  26-11月-19
 
 BS Key Type LV Size    Device Type Elapsed Time Completion Time
 ------- ---- -- ---------- ----------- ------------ ---------------
 253   Full  17.77M   DISK    00:00:00   26-11月-19  
     BP Key: 253  Status: AVAILABLE Compressed: NO Tag: TAG20191126T151703
     Piece Name: /home/oracle/rman_backup/c-1392946895-20191126-00
  SPFILE Included: Modification time: 26-11月-19
  SPFILE db_uniqe_name: ORCL
  Control File Included: Ckp SCN: 47696860   Ckp time: 26-11月-19
~~~

### 由上面的"list backup;" 输出可以看出，备份集中的文件内容是：

•       /home/oracle/rman*backup/dblv0*ORCL*20191120*d7uhb2ap*1*1.bak 是插接数据库PDBORCL的备份集

•       /home/oracle/rman*backup/dblv0*ORCL*20191120*d8uhb2c6*1*1.bak 是ORCL的备份集

•       /home/oracle/rman*backup/dblv0*ORCL*20191120*d9uhb2ei*1*1.bak是PDB$SEED的备份集

•       /home/oracle/rman*backup/arclv0*ORCL*20191120*dauhb2fm*1*1.bak是归档文件的备份集

•       /home/oracle/rman_backup/c-1392946895-20191120-01是参数文件(SPFILE)和控制文件(Control File)的备份集

### 10.4. 备份后修改数据库

~~~
[oracle@oracle-pc ~]$ sqlplus study/123@pdborcl
 SQL> create table t1 (id number,name varchar2(50));
 Table created.
 SQL> insert into t1 values(1,'zhang');
 1 row created.
 SQL> commit;
 Commit complete.
 SQL> select * from t1;
 
     ID NAME
 ---------- --------------------------------------------------
     1 zhang
 SQL> exit
~~~

<img src="images/14.png" alt="img" style="zoom:50%;" />

### 10.5. 删除数据库文件模拟数据损坏

~~~
[oracle@oracle-pc ~]$ rm /home/oracle/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf
~~~

<img src="images/15.png" alt="img" style="zoom:50%;" />

删除数据库文件后修改数据

 删除数据文件后，仍然可以增加一条数据。这是因为增加的数据并没有写入数据文件，而是写到了日志文件中。如果增加的数据较多的时候，就会出问题了。 

~~~
[oracle@oracle-pc ~]$ sqlplus study/123@pdborcl
 SQL> insert into t1 values(2,'wang');
 1 row created.
 SQL> commit;
 Commit complete.
 SQL> select * from t1;
     ID NAME
 ---------- --------------------------------------------------
     1 zhang
     2 wang
 SQL> 
~~~

<img src="images/16.png" alt="img" style="zoom:50%;" />

### 10.6. 数据库完全恢复

#### 10.6.1. 重启损坏的数据库到mount状态

通过shutdown immediate无法正常关闭数据库，只能通过shutdown abort强制关闭。然后将数据库启动到mount状态。

~~~
[oracle@oracle-pc ~]$ sqlplus / as sysdba
 SQL> shutdown immediate
 ORA-01116: 打开数据库文件 10 时出错
 ORA-01110: 数据文件 10: '/home/oracle/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf'
 ORA-27041: 无法打开文件
 Linux-x86_64 Error: 2: No such file or directory
 Additional information: 3
 SQL> shutdown abort
 ORACLE instance shut down.
 SQL> startup mount
 ORACLE instance started.
 
 Total System Global Area 1577058304 bytes
 Fixed Size         2924832 bytes
 Variable Size       738201312 bytes
 Database Buffers     654311424 bytes
 Redo Buffers        13848576 bytes
 In-Memory Area      167772160 bytes
 Database mounted.
~~~

<img src="images/16.png" alt="img" style="zoom:50%;" />

#### 10.6.2. 开始恢复数据库

~~~
[oracle@oracle-pc ~]$ rman target /
 RMAN> restore database ;
 Starting restore at 20-11月-19
 using target database control file instead of recovery catalog
 allocated channel: ORA_DISK_1
 channel ORA_DISK_1: SID=243 device type=DISK
 
 skipping datafile 5; already restored to file /home/oracle/app/oracle/oradata/orcl/pdbseed/system01.dbf
 skipping datafile 7; already restored to file /home/oracle/app/oracle/oradata/orcl/pdbseed/sysaux01.dbf
 channel ORA_DISK_1: starting datafile backup set restore
 channel ORA_DISK_1: specifying datafile(s) to restore from backup set
 channel ORA_DISK_1: restoring datafile 00008 to /home/oracle/app/oracle/oradata/orcl/pdborcl/system01.dbf
 channel ORA_DISK_1: restoring datafile 00009 to /home/oracle/app/oracle/oradata/orcl/pdborcl/sysaux01.dbf
 channel ORA_DISK_1: restoring datafile 00010 to /home/oracle/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf
 channel ORA_DISK_1: restoring datafile 00011 to /home/oracle/app/oracle/oradata/orcl/pdborcl/example01.dbf
 channel ORA_DISK_1: restoring datafile 00012 to /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_1.dbf
 channel ORA_DISK_1: restoring datafile 00013 to /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_2.dbf
 channel ORA_DISK_1: restoring datafile 00016 to /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_3.dbf
 channel ORA_DISK_1: restoring datafile 00017 to /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_4.dbf
 channel ORA_DISK_1: restoring datafile 00077 to /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users03_1.dbf
 channel ORA_DISK_1: restoring datafile 00078 to /home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users03_2.dbf
 channel ORA_DISK_1: reading from backup piece /home/oracle/rman_backup/dblv0_ORCL_20191120_d7uhb2ap_1_1.bak
 channel ORA_DISK_1: piece handle=/home/oracle/rman_backup/dblv0_ORCL_20191120_d7uhb2ap_1_1.bak tag=TAG20191120T083953
 channel ORA_DISK_1: restored backup piece 1
 channel ORA_DISK_1: restore complete, elapsed time: 00:01:41
 channel ORA_DISK_1: starting datafile backup set restore
 channel ORA_DISK_1: specifying datafile(s) to restore from backup set
 channel ORA_DISK_1: restoring datafile 00001 to /home/oracle/app/oracle/oradata/orcl/system01.dbf
 channel ORA_DISK_1: restoring datafile 00003 to /home/oracle/app/oracle/oradata/orcl/sysaux01.dbf
 channel ORA_DISK_1: restoring datafile 00004 to /home/oracle/app/oracle/oradata/orcl/undotbs01.dbf
 channel ORA_DISK_1: restoring datafile 00006 to /home/oracle/app/oracle/oradata/orcl/users01.dbf
 channel ORA_DISK_1: reading from backup piece /home/oracle/rman_backup/dblv0_ORCL_20191120_d8uhb2c6_1_1.bak
 channel ORA_DISK_1: piece handle=/home/oracle/rman_backup/dblv0_ORCL_20191120_d8uhb2c6_1_1.bak tag=TAG20191120T083953
 channel ORA_DISK_1: restored backup piece 1
 channel ORA_DISK_1: restore complete, elapsed time: 00:01:55
 Finished restore at 20-11月-19
 
 RMAN> recover database;
 RMAN> alter database open;
 Statement processed
 RMAN> exit
~~~

<img src="images/17.png" alt="img" style="zoom:50%;" />

#### 10.6.3. 查询数据是否恢复

~~~
[oracle@oracle-pc ~]$ sqlplus study/123@pdborcl
 SQL> select * from t1;
     ID NAME
 ---------- --------------------------------------------------
     1 zhang
     2 wang
 SQL> 
~~~

<img src="images/19.png" alt="img" style="zoom:50%;" />

 

 

 由以上查询结果可见，数据100%恢复了! 

## 11. DataGuard实现数据库整体的异地备份

 Data Guard 是Oracle的集成化灾难恢复解决方案，该技术可以维护生产数据库一个或多个同步备份，由一个主数据库和多个备用数据库组成，并形成一个独立的、易于管理的数据保护方案。
Data Guard 备用数据库可以与主系统位于相同的数据中心，也可以是在地理位置上分布较远的远程灾难备份中心。

   Data Guard 的基本原理是，当某次事务处理对生产数据库中的数据做出更改时，Oracle 数据库将在一个联机重做日志 文件中记录此次更改。在DataGuard中可以配置写日志的这个过程，除了把日志记录到本地的联机日志文件和归档日志文件中，还可以通过网络，把日志信息发送的远程的备用数据库服务器上。这个备用日志文件写入过程可以是实时，同步的，以实现零数据丢失(最大保护模式);也可以是异步的，以减少对网络带宽的压力(最大可用性模式);或者是通过归档日志文件、一个日志文件的批量传输模式，以减少对生产系统的性能影响(最大性能模式)。当备份数据库接收到日志信息后，Data Guard 可以自动利用日志信息实现数据的同步。当主数据库打开并处于活动状态时，备用数据库可以执行恢复操作，如果主数据库出现了故障，备用数据库即可以被激活并接管生产数据库的工作。

  安装配置Data Guard的步骤如下:

(1)将主数据库设置成归档状态和自动归档模式

(3)在主数据库 上创建备用数据库控制文件、初始化参数文件:

(4)把将主数据库备份文件、控制文件和初始化参数文件复制到备用数据库上;

(5)设置 备用数据库控制文件、初始化参数文件和备用数据库侦听;

(6) 设置备用数据库连接主数据库的服务名与主数据库到备用数据库的服务名;

(7)启动备用数据库实例， 初始化日 志应用服务并且将备用数据库设置成standby;

(8)启动归档到备用数据库。

   四、Data Guard在我院HIS中的应用

   (1)容灾备份: 我院在住院大楼的机房新建了-个异地备份机房，硬件采用一台IBM X3650服务器， 服务器上配置5块146G硬盘并做了RAID5,安装配置了Oracle 10g 和Data Guard. 保证了无论是硬盘或网络出现故障的情况下全院信息系统能正常运行。同时对于火灾、盗窃等非技术情况下的异地备份，提高了整体系统完全性。经过多次测试，主数据库和备份数据库之间能够在5分钟之类切换，基本上能保证系统的连续运行。

   Data Guard作为Oracle免费推出的一个灾难恢复技术，利用Oracle 10g Data Guard而实现备用数据库操作的好处可以简单地总结为:它提供了灾难保护并防止数据丢失、维护主数据库的几个事务一致的副本、 防止灾难、数据损坏和用户错误、无需昂贵且复杂的HW/SW镜像。但是Data Guard也有以下缺点:不支持异构、不可跨平台、数据库版本需一致， 目标数据库出入rcovery状态不可用等，如果用户需要更高性能及可靠性可以购买第三方软件，例如DSG Real Synce等产品。

 第一步：备库

~~~
mkdir -p /home/oracle/app/oracle/diag/orcl
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdborcl
mkdir -p /home/oracle/arch
mkdir -p /home/oracle/rman
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdbseed/
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdb/

//linux 命令，一行一行在终端运行
//删除原有数据库:
$sqlplus / as sysdba         //连数据库
shutdown immediate;          //sql>
startup mount exclusive restrict;      //sql>
drop database;              //sql>
exit;                    //sql>
启动到nomount
$sqlplus / as sysdba         //连数据库
startup nomount             //sql>
sql>
//换到primary
~~~

<img src="images\20.png" alt="image-20191126204925927" style="zoom:50%;" />



![](images\21.png)

第二步：主库:

~~~
$sqlplus /  sysdba              //sql>
select group#,thread#,members,status from v$log;          //sql>

alter database add standby logfile  group 5 '/home/oracle/app/oracle/oradata/orcl/stdredo1.log' size 50m; //sql>
alter database add standby logfile  group 6 '/home/oracle/app/oracle/oradata/orcl/stdredo2.log' size 50m; //sql>
alter database add standby logfile  group 7 '/home/oracle/app/oracle/oradata/orcl/stdredo3.log' size 50m; //sql> 
alter database add standby logfile  group 8 '/home/oracle/app/oracle/oradata/orcl/stdredo4.log' size 50m; //sql>
~~~

主库环境开启强制归档

~~~
ALTER DATABASE FORCE LOGGING;  //sql>

alter system set LOG_ARCHIVE_CONFIG='DG_CONFIG=(orcl,stdorcl)' scope=both sid='*';         //sql>
alter system set log_archive_dest_1='LOCATION=/home/oracle/arch VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=orcl' scope=spfile;
alter system set LOG_ARCHIVE_DEST_2='SERVICE=stdorcl LGWR ASYNC  VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=stdorcl' scope=both sid='*';
alter system set fal_client='orcl' scope=both sid='*';    
alter system set FAL_SERVER='stdorcl' scope=both sid='*';  
alter system set standby_file_management=AUTO scope=both sid='*';
alter system set DB_FILE_NAME_CONVERT='/home/oracle/app/oracle/oradata/stdorcl/','/home/oracle/app/oracle/oradata/orcl/' scope=spfile sid='*';  
alter system set LOG_FILE_NAME_CONVERT='/home/oracle/app/oracle/oradata/stdorcl/','/home/oracle/app/oracle/oradata/orcl/' scope=spfile sid='*';
alter system set log_archive_format='%t_%s_%r.arc' scope=spfile sid='*';
alter system set remote_login_passwordfile='EXCLUSIVE' scope=spfile;
alter system set PARALLEL_EXECUTION_MESSAGE_SIZE=8192 scope=spfile;

exit;
~~~


编辑主库以及备库的/home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora文件

~~~
$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora  //terminal

ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.206.131)(PORT = 1521))  //**
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )

stdorcl =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.206.132)(PORT = 1521))  //**
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SID = orcl)
    )
  )

  //ifconfig 查询ip后 测试两能否平ping通  只改ip地址   最后一个块被替换为以上两个块  记得保存（主+备）
~~~

在主库上生成备库的参数文件:

~~~
$sqlplus /  as sysdba   //如果没有出现sql>
SQL>create pfile from spfile;
生成/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora
exit;
~~~

<img src="images\22.png" alt="image-20191126205058301" style="zoom:50%;" />

将主库的参数文件，密码文件拷贝到备库:

scp /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora 192.168.206.132:/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/
scp /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/orapworcl 192.168.206.132:/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/

将主库复制到备库

~~~
$rman target sys/123@orcl auxiliary sys/123@stdorcl      //terminal运行
rman>
执行duplicate:
run{ 
allocate channel c1 type disk;
allocate channel c2 type disk;
allocate channel c3 type disk;
allocate AUXILIARY channel c4 type disk;
allocate AUXILIARY channel c5 type disk;
allocate AUXILIARY channel c6 type disk;
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  NOFILENAMECHECK;
release channel c1;
release channel c2;
release channel c3;
release channel c4;
release channel c5;
release channel c6;
}
~~~


//等待直至出现rman>

第三步：备库

在备库上更改参数文件

$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora  //运行

文件内容是：

~~~
orcl.__data_transfer_cache_size=0
orcl.__db_cache_size=671088640
orcl.__java_pool_size=16777216
orcl.__large_pool_size=33554432
orcl.__oracle_base='/home/oracle/app/oracle'#ORACLE_BASE set from environment
orcl.__pga_aggregate_target=536870912
orcl.__sga_target=1258291200
orcl.__shared_io_pool_size=50331648
orcl.__shared_pool_size=301989888
orcl.__streams_pool_size=0
*._allow_resetlogs_corruption=TRUE
*._catalog_foreign_restore=FALSE
*.audit_file_dest='/home/oracle/app/oracle/admin/orcl/adump'
*.audit_trail='db'
*.compatible='12.1.0.2.0'
*.control_files='/home/oracle/app/oracle/oradata/orcl/control01.ctl','/home/oracle/app/oracle/fast_recovery_area/orcl/control02.ctl','/home/oracle/app/oracle/fast_recovery_area/orcl/control03.ctl'
*.db_block_size=8192
*.db_domain=''
*.db_file_name_convert='/home/oracle/app/oracle/oradata/orcl/','/home/oracle/app/oracle/oradata/stdorcl/'
*.db_name='orcl'
*.db_unique_name='stdorcl'
*.db_recovery_file_dest='/home/oracle/app/oracle/fast_recovery_area'
*.db_recovery_file_dest_size=4823449600
*.diagnostic_dest='/home/oracle/app/oracle'
*.dispatchers='(PROTOCOL=TCP)(dispatchers=1)(pool=on)(ticks=1)(connections=500)(sessions=1000)'
*.enable_pluggable_database=true
*.fal_client='stdorcl'
*.fal_server='orcl'
*.inmemory_max_populate_servers=2
*.inmemory_size=157286400
*.local_listener=''
*.log_archive_config='DG_CONFIG=(stdorcl,orcl)'
*.log_archive_dest_1='LOCATION=/home/oracle/arch VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=stdorcl'
*.log_archive_dest_2='SERVICE=orcl LGWR ASYNC  VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=orcl'
*.log_archive_format='%t_%s_%r.arc'
*.log_file_name_convert='/home/oracle/app/oracle/oradata/orcl/','/home/oracle/app/oracle/oradata/stdorcl/'
*.max_dispatchers=5
*.max_shared_servers=20
*.open_cursors=400
*.parallel_execution_message_size=8192
*.pga_aggregate_target=511m
*.processes=300
*.recovery_parallelism=0
*.remote_login_passwordfile='EXCLUSIVE'
*.service_names='ORCL'
*.sga_max_size=1572864000
*.sga_target=1258291200
*.shared_server_sessions=200
*.standby_file_management='AUTO'
*.undo_tablespace='UNDOTBS1'
~~~

//以上全部替换弹出的文件内容    记得保存

<img src="images\23.png" alt="image-20191126205030287" style="zoom:50%;" />

在备库增加静态监听

~~~
$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/listener.ora //运行

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (ORACLE_HOME = /home/oracle/app/oracle/product/12.1.0/db_1)
      (SID_NAME = orcl)
    )
  )
 //添加到此文件最后 
//重新启动,备库开启实时应用模式:：
$sqlplus / as sysdba //再次连数据库
shutdown immediate   //sql>
startup              //sql>
alter database recover managed standby database disconnect;        //sql>
~~~

测试：

在主库连数据库

~~~
$sqlplus / as sysdba //连数据库
create table lft(sex varchar);
~~~

进入备库：

~~~
$sqlplus / as sysdba //连数据库
select * from lft;
~~~


