## MySQL 补充

## 1. 数据库引擎

### 1.1 InnoDB 和 MyiSAM

|              | MyISAM | InnoDB         |
| ------------ | ------ | -------------- |
| 事务         | 不支持 | 支持           |
| 数据行锁定   | 不支持 | 支持           |
| 外键约束     | 不支持 | 支持           |
| 全文索引     | 支持   | 不支持         |
| 表空间的大小 | 较小   | 较大，约为两杯 |

* MYISAM 节约空间，速度较快
* INNODB 安全性高，事务处理，多表多用户



### 1.2 物理引擎物理文件上的区别

* innoDB数据库表中只有一个*.frm 文件，以及上级目录下的 ibdata1 文件
* MyISAM 对于文件
  * *.frm 表结构的定义文件
  * *.MYD  数据文件（data）
  * *.MYI 索引文件（index）

## 2. 数据管理

### 2.1 外键（现在不用了）

### 2.2 DML语言 CUD

### 2.3 DQL Read

#### 2.3.1 别名

```sql
select `id` as studen_id, `username` as name from users as user; -- 这里别名不需要加``
```

#### 2.3.2 concat

```sql
select concat('name:' ,username) as 名字 from users; --拼接的地方要用‘’不是``
```

#### 2.3.3 distinct

去重

```sql
select distinct * from users; -- 去除重复的
```

#### 2.3.4 逻辑运算

```sql
select `studentNo`, `studentResult` + 1 as '提分后' from result; --所有成绩+1显示
```

#### 2.3.5 模糊查询

like  （% 代表可以无所谓几个字符，_代表只能有一个）

In （set） 集合查询

NULL

#### 2.3.5 连表查询

##### 2.3.5.1 三种基础 join

| 操作       | 描述                                       |
| ---------- | ------------------------------------------ |
| Inner join | 如果表中至少有一个匹配,就返回行            |
| Left join  | 即使右表没有匹配，也会从左表中返回所有的值 |
| right join | 即使左表中没有匹配，也会从右表中返回。     |

##### 2.3.5.2 自己理解

案例：student 和 score 两张表，需要查询学生信息和考试信息

student：studentNo, studentName.

Score: studentNo,subjectNo, result

* inner相当于并，那么只要有一方有那么就会满足所以就会返回行

  ```sql
  select s.studentNo, studentName, subjectNo, studentResult
  form student as s
  inner join result as r
  on s.studentNo = r.studentNo;
  ```

* Right join 相当于 右表做主表无论如何都会查询出来 左表如果有就有 没有就NULL

  ```sql
  select s.studentNo, studentName, subjectNo, studentResult
  form student as s
  right join result as r
  on s.studentNo = r.studentNo;
  ```

* left join 类似

##### 2.3.5.3 自连接（了解）

* **包含**父子关系的一张表**拆成**一张表示父子关系的表
* 把一张表用两次

```sql
select a.`categoryName` as '父栏目', b.`categoryName` as '子栏目'
from `category` as a, `category` as b 
where a.`categoryid` = b.`pid`;
```

#### 2.3.6 排序

```sql
xxxxx order by `字段` ASC --或者DSC
```

#### 2.3.7 分页

```sql
xxxxx limit 0, 5 --起始数据index + 每页的数目
```

#### 2.3.8 子查询

嵌套查询对比

![image-20200423203333766](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200423203333766.png)



#### 2.3.8 函数 

聚合函数 那些逻辑类型的具体查文档

```sql
count(*) count(1) count(col)
--不忽略null 忽略null 都是查询所有   查询指定列会忽略null
xxx group by `字段` --通过某个字段分组
```



## 3. 索引

本质是数据结构

* 主键索引	(PRIMARY KEY) 唯一索引，**不可重复**只能有一个列做主键
* 唯一索引   (UNIQUE KEY) 避免重复列的出现，唯一索引**可以重复**，多个列都可以标示为唯一索引
* 常规索引 （KEY/INDEX）

### 3.1 基础使用

```sql
show index from student; --查看索引
alter table school.student add fulltext index `studentName` (`studentName`); 
--增加一个全文索引 索引名 + 列名
explain select * from student; --非全文搜索
explain select * from student where match(studentName) against('刘');
```

### 3.2 创建索引规则

```sql
id_表名_字段名
```

```sql
alter table school.student add fulltext index `studentName` (`studentName`); 
--1.增加一个全文索引 索引名 + 列名
create index id_app_user_name on app_user(`name`);
--2.id_app_user_name 索引名字 app_user(`name`) 表（字段）
```



## 4. 数据库连接池

### 4.1 DBCP

### 4.2 C3P0

## 5. 隔离级别（补充）

针对事务可能产生的问题

### 5.1 脏读(dirty read) 一个事务读取了另一个事务尚未提交的数据

* 只读了一次！但是读到的是可能未完成的数据

当一个事务正在多次修改某个数据，而在这个事务中这多次的修改都还未提交，这时一个并发的事务来访问该数据，就会造成两个事务得到的数据不一致。例如：用户A向用户B转账100元，对应SQL命令如下

### 5.2 不可重复读(non-repeatable read)当一个事务多次读取时，发现数据有变化

* 多次查询 中间其他事务提交了新数据 所以查询结果不同

不可重复读是指在对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询间隔，被另一个事务修改并提交了

例如事务T1在读取某一数据，而事务T2立马修改了这个数据并且提交事务给数据库，事务T1再次读取该数据就得到了不同的结果，发送了不可重复读。

不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。

### 5.3 幻读(phantom read) 一个事务的操作导致另一个事务前后两次查询的结果数据量不同。

* 这个是查询多条数据的时候 上面是查询一条数据

例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这时事务T2又对这个表中插入了一行数据项，而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发现还有一行没有修改，其实这行是从事务T2中添加的，就好像产生幻觉一样，这就是发生了幻读。



作者：凯哥学堂
链接：https://www.jianshu.com/p/8c44e9bf2ac2
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## A 指令

需要注意的是 用 **``** 不要用**''**

### 1. 修改表

```sql
alter table teacher rename as teacher1; --修改表名字
alter table teacher add age int(11); --添加属性
alter table teacher modify age varchar(11); --修改约束 
alter table teacher change age age1 --修改属性名字 这个也可以使修改约束 只需要添加在新的名字和后面
alter table teacher drop age --删除字段
drop table if exists teacher -- 如果存在删除
```

### 2. 删库

```sql
turncate `student`
```

相同点：delete 和 turncate都**不会清除表结构**

不同点：turncate **自增清零**，并且**不影响事务**

### 3. 开启事务

```sql
set autocommit = 0;
start transaction
...
...
commit; 
```

