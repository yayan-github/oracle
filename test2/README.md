# oracle实验二：用户及权限管理

##### 姓名：陈奕妍       										学号：201810414101 										班级：软工一班 

##  实验目的

掌握用户管理、角色管理、权根维护与分配的能力，掌握用户之间共享对象的操作技能。

## 实验内容

Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：

* 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有con_res_view的用户就同时拥有这三种权限。
* 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。
* 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。

### 1、以system登录到pdborcl，创建角色con_res_cyy和用户cyy，并授权和分配空间。

```sql
$ sqlplus system/123@pdborcl
SQL> CREATE ROLE con_res_cyy;
Role created.
SQL> GRANT connect,resource,CREATE VIEW TO con_res_cyy;
Grant succeeded.
SQL> CREATE USER cyy IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
User created.
SQL> ALTER USER cyy QUOTA 50M ON users;
User altered.
SQL> GRANT con_res_cyy TO cyy;
Grant succeeded.
SQL> exit
```

> 语句“ALTER USER CYY QUOTA 50M ON users;”是指授权CYY用户访问users表空间，空间限额是50M。

![pic1](.\pic1.png)

### 2、新用户cyy连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。

```sql
$ sqlplus new_user/123@pdborcl
SQL> show user;
USER is "NEW_USER"
SQL> CREATE TABLE mytable (id number,name varchar(50));
Table created.
SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');
1 row created.
SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');
1 row created.
SQL> CREATE VIEW myview AS SELECT name FROM mytable;
View created.
SQL> SELECT * FROM myview;
NAME
--------------------------------------------------
zhang
wang
SQL> GRANT SELECT ON myview TO hr;
Grant succeeded.
SQL>exit
```



![pic2](.\pic2.png)

### 3、用户hr连接到pdborcl，查询cyy授予它的视图myview.

```
$ sqlplus hr/123@pdborcl
SQL> SELECT * FROM new_user.myview;
NAME
--------------------------------------------------
zhang
wang
SQL> exit
```



![pic3](.\pic3.png)

### 4、测试一下同学用户之间的表的共享，只读共享和读写共享都测试一下。

![pic4](.\pic4.png)

### 5、查看数据库的使用情况

#### 1）查看表空间的数据库文件

```
$ sqlplus system/123@pdborcl

SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

```

![pic5](.\pic5.png)

#### 2) 每个文件的磁盘占用情况

```
SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
```

![pic6](.\pic6.png)

* autoextensible是显示表空间中的数据文件是否自动增加。

* MAX_MB是指数据文件的最大容量。

  











