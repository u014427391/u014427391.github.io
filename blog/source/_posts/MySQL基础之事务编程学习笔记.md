
title: MySQL基础之事务编程学习笔记

date: 2019-12-03 00:00:00
---
<Excerpt in index | 首页摘要> 

在学习《MySQL技术内幕：SQL编程》一书，并做了笔记。本博客内容是自己学了《MySQL技术内幕：SQL编程》事务编程一章之后，根据自己的理解做的笔记，内容和书本并不一致，不过书本实验都经过自己验证，基于MySQL5.7版本。做笔记的目的是方便自己复习，同时分享出来或许对其他人或许有点帮助


<!-- more -->
<The rest of contents | 余下全文>

MySQL基础之事务编程学习笔记

在学习《MySQL技术内幕：SQL编程》一书，并做了笔记。本博客内容是自己学了《MySQL技术内幕：SQL编程》事务编程一章之后，根据自己的理解做的笔记，内容和书本并不一致，不过书本实验都经过自己验证，基于MySQL5.7版本。做笔记的目的是方便自己复习，同时分享出来或许对其他人或许有点帮助


## 1、事务概述
事务是数据库区别于文件系统的重要特性之一，提到事务肯定会想到事务的4个特性ACID，要保证业务的正常使用，必须保证ACID，ACID表示原子性(atomicity)、一致性(consistency)、隔离性(isolation)、持久性(durability)，一个运行良好的事务系统也是要求具备这些特征

* 原子性(atomicity)：一个事务必须被视为一个不可分割的最小工作单位，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，不能只执行一部分操作

* 一致性(consistency)：一致性要求数据库总是从一个一致性的状态转换为另外一个一致性的状态，比如银行转账的例子，一个用户转账账号里减了200元，另外一个收账账号必须增加，一个失败数据必须全部回退状态，要保证事务一致性

* 隔离性(isolation)：一般来说，一个事务所做的修改在提交之前对其它事务来说都是不可见的

* 持久性(durability)：事务一旦提交，所做的状态就会永久保存在数据库中，即使系统奔溃，修改的数据也不会丢

## 2、事务分类
《MySQL技术内幕：SQL编程》一书中有提到了事务的分类，里面介绍了事务的分类，从理论角度，将事务分为：
* 扁平事务(flat transactions)
* 带有保存点的扁平事务(flat transactions with savepoints)
* 链事务(chained transactions)
* 嵌套事务(nested transactions)
* 分布式事务(distributed transactions)
**扁平事务**，这种事务是最基本，也是最简单的一种事务，其特点就是所有的操作都处于同一个层次，所有操作都处于begin work和commit work(or rollback work)之间，具有事务的原子性，不过扁平事务是不能提交或回滚事务的某一部分，或者分几个部分提交，因为扁平事务本来就是一起提交，或者一起回滚
**带有保存点的扁平事务**，因为扁平事务的局限性，不能支持部分事务回滚，所以就有了带保存点的扁平事务，这种事务能通过savepoint命令保存一个保存点，需要rollback时候，使用命令`ROLLBACK TO SAVEPOINT identifier;`需要注意的是这种事务保存点并非具有持久性，rollback to savepoint id之后并不表示事务已经真的回滚，需要执行rollback work或者commit work才算真正的提交事务
**链事务**，前面介绍了，带有保存点的扁平事务保存点不具有事务的持久性，假如系统出现奔溃的情况，那数据不是都没保存？也不能回滚到最近的保存点，所以就有了链事务的出现，链事务能做出现系统奔溃的情况，回滚到最近的保存点，当然在mysql系统中默认是不开启的，链事务是作用是，在提交一个事务时，释放不需要的数据对象，将必要的上下文隐式传给下一个开始的事务，*也就是将提交事务操作和开始下一个事务操作合并为一个原子操作*，引用书中的图示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191218111601113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191218111639352.png)
链事务虽然可以避免系统奔溃的情况，数据丢失的情况，但是链事务是不支持回滚到指定保存点的，只能回退数据到上一个事务，不过带保存点事务能做到回滚到指定保存点

**嵌套事务**，嵌套事务从名称就可以理解，这种一种层次结构的嵌套事务，有一个顶级事务(top-level transaction)控制着各个子事务(subtransaction)，这种子事务可以是扁平事务、带保存点的扁平事务、链事务等等
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191218112759591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
引用书中的嵌套式事务特征
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191218114530216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
可以归纳重点，这种嵌套事务其实可以理解为一种树形结构，分为顶级事务，子事务，整个事务需要顶级事务提交后才真正算提交，任何一个子事务回滚都会导致所有子事务一起回滚，所有说整个嵌套事务具有原子性、一致性、隔离性，但是不具有事务持久性

**分布式事务**，分布式很容易理解，这种是在分布式环境运行的事务，一般也是扁平事务，不过是分布式环境的

## 3、事务控制语句
有了前面的理解之后，就可以学习事务的控制语句了，实践之后就可以更好地理解前面的理论知识

mysql默认事务是自动提交的，不过是可以通过命令进行关闭，也可以使用事务控制语句开启一个事务，所以，归纳一下，方法有二
```sql
## 方法1：通过set autocommit = 0关闭事务自动提交
# 查询是否开启事务自动提交，1：事务自动提交；2：事务不自动提交
SELECT @@autocommit;
# 关闭事务自动提交
SET autocommit = 0;
## 方法2：通过BEGIN 或者 START TRANSACTION命令
# 开启一个事务，使用这个命令之后，事务要使用commit或者commit work命令才会提交
BEGIN;
# START TRANSACTION命令和Begin命令效果一样 
START TRANSACTION;
```


事务控制语句语法：

* BEGIN; / START TRANSACTION;
这两个命令效果一样，都是用于开启事务
* COMMIT WORK; / COMMIT; 
Commit和Commit work在默认情况效果是一样的，在链事务里就不一样，Commit还是执行Commit操作，Commit work在链事务里是执行Commit和开启另外一个新的事务，相当于commit+ BEGIN / START TRANSACTION，**这两个操作是原子操作**
* ROLLBACK WORK;/ ROLLBACK; 
 ROLLBACK和ROLLBACK WORK效果是一样的，都是回滚事务
* SAVEPOINT identifier(保存点id命名);  
SAVEPOINT命令用于使用保存点，也就是前面介绍的带保存点的事务
eg:`savepoint tp1`
* RELEASE SAVEPOINT identifier;
这个命令用于删除保存点
* ROLLBACK TO SAVEPOINT identifier;
这个命令用于回滚到对应的保存点



Mysql completion_type介绍：
```sql
# 查询completion_type类型
SHOW VARIABLES LIKE 'completion_type';
# 也可以使用@@系统变量方式查询
SELECT @@completion_type;
## completion_type默认值为0，它还有1和2两种值，不过在新版mysql5.7测试，发现值为2的情况不起效
SET @@completion_type = 1;# commit+chain(链事务，提示事务之后还会start transaction)
SET @@completion_type = 2;# commit+release(经过测试，在新版mysql5.7这个设置不起效，以前版本开启之后是会关闭Session的)
```
下面举例实验：
```sql
# 新建一个测试表
CREATE TABLE t (a INT,PRIMARY KEY(a))ENGINE = INNODB;
# 开启链事务
SET @@completion_type = 1;
# 开启一个事务，如下两个命令皆可
BEGIN; START TRANSACTION;
# 新增一条数据
INSERT INTO t SELECT 1;
# commit work在@@completion_type = 1的情况相关于commit+start transaction
COMMIT WORK;
# 新增一条数据
INSERT INTO t SELECT 2;
# 新增一条重复数据，让数据库报错
INSERT INTO t SELECT 2;
# 查询，有两条数据
SELECT * FROM t;
# 回滚事务
ROLLBACK;
# 再次查询，发现只有1这条数据，由此说明COMMIT WORK在commit之后，还开启了一个事务，当然是在autocommit值为1的情况进行验证的
SELECT * FROM t;

# completion_type=2在旧版估计才有用，我在5.7版本验证，发现并不起作用
SET @@completion_type=2;
# 开启事务
BEGIN;
# 新增一条数据
INSERT INTO t SELECT 1;
# commit work相关于commit+release，这个操作会关闭session
COMMIT WORK;
# 使用@@version验证会话是否关闭
SELECT @@version;
```
书中mysql版本是5.1，commit work之后是会关闭会话的，不过在5.7还是可以查询的，5.1版本报错引用书中图片
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200101115850490.png)


## 4、事务隔离级别
SQL标准中定义四种隔离级别，每种存储引擎实现的隔离级别是不同的

* READ UNCOMMITTED(未提交读)
在READ UNCOMMITTED级别，事务即使没提交，对其它事务也是可见，允许事务读取未提交的数据，这也被称为脏读(Dirty Read)，所以在实际生产中很少用

* READ COMMIT(提交读)
大部分数据库系统的默认隔离级别都是READ COMMIT，很明显这种隔离级别只能读取到已经提交事务的数据

* REPEATABLE READ(可重复读)
REPEATABLE READ可以解决脏读的问题，不过不能解决幻读问题，所谓幻读是指，在读取某个范围数据时候，另外一个事务又向这个范围写数据了，当之前事务再次读取这个范围数据时候就会产生幻读

* SERIALIZABLE(可串行化)
SERIALIZABLE是最高的隔离级别，SERIALIZABLE强制事务串行执行，可以避免幻读问题，其实保证SERIALIZABLE是常用行锁的方式来保证一致性的，SERIALIZABLE会在每一行的数据上都加上锁，SERIALIZABLE虽然可以保证事务安全性，不过服务器性能不好也会导致锁争用问题

查看当前会话隔离级别
```sql
SELECT @@tx_isolation;
```
查看系统全局隔离级别
```sql
SELECT @@global.tx_isolation;
```
设置当前会话隔离级别：
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```
设置mysql系统全局隔离级别：
```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

## 5、分布式事务
MySQL中的分布式事务是通过XA事务事项的，当然要支持事务的存储引擎才可以，比如InnoDB，MySQL5.7版本，大部分存储引擎比如MyISAM，都是不支持事务的，也显然不支持XA事务，存储引擎不熟悉可以参考我之前博客:[MySQL架构之逻辑架构简介](https://blog.csdn.net/u014427391/article/details/100170265)

提到分布式事务，比较常见的就是银行转账的例子，对于银行银行转账这种场景就肯定要保证事务一致性，引用《MySQL技术内幕：SQL编程》书中例子，上海用户david向北京用户mariah的存储卡转账1000元，转账过程和到账过程肯定是分开的，首先上海银行的数据库先执行账号金额update操作，接着北京银行的数据库再执行账号金额update，假如没有保证事务一致性，上海银行数据库执行了，事务提交了，然后北京用户数据库没进行事务提交，这种情况是不允许的，所以就要保证事务一致性

在数据库层面可以使用mysql的XA事务，xa事务可以支持多种数据库，xa事务由一个或多个资源管理器(resource manager)、一个事务管理器(transaction manager)以及一个应用程序组成(application program)

* 应用程序：定义事务的边界，指定全局事务中的操作
* 资源管理器：提供访问事务资源的方法，通常一个数据库就是一个资源管理器
* 事务管理器：协调全局事务，需要和参与到全局事务的资源管理器进行通信
引用《MySQL技术内幕：SQL编程》一书的图示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191217191317307.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
XA基本用法可以参考[MySQL官方文档](https://dev.mysql.com/doc/refman/5.7/en/xa.html)，从官网复制的xa用法：
```sql
XA {START|BEGIN} xid [JOIN|RESUME]

XA END xid [SUSPEND [FOR MIGRATE]]

XA PREPARE xid

XA COMMIT xid [ONE PHASE]

XA ROLLBACK xid

XA RECOVER [CONVERT XID]
```
单机环境XA分布式例子：
```sql
mysql>XA START 'a';
mysql>INSERT INTO t SELECT 1;
mysql>XA END 'a';
mysql>XA PREPARE 'a';
mysql>XA RECOVER;
mysql>XA COMMIT 'a';
```
当然XA事务肯定是应用于分布式环境，所以上面例子只是练习熟悉一下基本命令，引用书中的例子进行实践，利用JDK的JTA进行分布式事务实现

建表：
```sql
create table t (username varchar(20),money float)engine=innodb;
insert into t(username,money) values('david',40000);
insert into t(username,money) values('jack',40000);
```
至于分布式事务，一般是通过应用层语言开发实现的，JDK里有提供了JTA(JAVA  Transaction API)，代码例子：
```java

import javax.transaction.xa.Xid;

public class JTAXid implements Xid {
    private int formatId;
    private byte gtrid[];
    private byte bqual[];

    public JTAXid(){}

    public JTAXid(int formatId, byte gtrid[], byte bqual[]){
        this.formatId = formatId;
        this.gtrid = gtrid;
        this.bqual = bqual;
    }

    @Override
    public int getFormatId() {
        return formatId;
    }

    @Override
    public byte[] getGlobalTransactionId() {
        return gtrid;
    }

    @Override
    public byte[] getBranchQualifier() {
        return bqual;
    }
}

```

```java

import com.mysql.jdbc.jdbc2.optional.MysqlXADataSource;

import javax.sql.XAConnection;
import javax.transaction.xa.XAResource;
import javax.transaction.xa.Xid;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

public class JTADemo {
	//数据源设置
    public static MysqlXADataSource getDataSource(String connString,String user,String pwd){
        try{
            MysqlXADataSource dataSource = new MysqlXADataSource();
            dataSource.setUrl(connString);
            dataSource.setUser(user);
            dataSource.setPassword(pwd);
            return dataSource;
        }catch (Exception e){
            e.fillInStackTrace();
            return null;
        }
    }

    public static void main(String[] args){
        String connStr1 = "jdbc:mysql://192.168.0.11:3306/bank_shanghai";
        String connStr2 = "jdbc:mysql://192.168.0.12:3306/bank_beijing";
        XAResource xaResource1=null,xaResource2=null;
        Statement statement1 =null,statement2 =null;
        try {
            MysqlXADataSource dataSource1 = getDataSource(connStr1, "root", "zaq12wsx");
            XAConnection xaConnection1 = dataSource1.getXAConnection();
            xaResource1 = xaConnection1.getXAResource();
            Connection connection1 = xaConnection1.getConnection();
            statement1 = connection1.createStatement();

            MysqlXADataSource dataSource2 = getDataSource(connStr2, "root", "zaq12wsx");
            XAConnection xaConnection2 = dataSource2.getXAConnection();
            xaResource2 = xaConnection2.getXAResource();
            Connection connection2 = xaConnection2.getConnection();
            statement2 = connection2.createStatement();
        }catch (SQLException e){
            e.printStackTrace();
        }

        try{

            Xid xid1 = new JTAXid(101,new byte[]{0x01},new byte[]{0x02});
            Xid xid2 = new JTAXid(101,new byte[]{0x11},new byte[]{0x12});

            xaResource1.start(xid1, XAResource.TMNOFLAGS);
            statement1.execute("update t set money = money-1000 where user='david'");
            xaResource1.end(xid1, XAResource.TMSUCCESS);

            xaResource2.start(xid2, XAResource.TMNOFLAGS);
            statement2.execute("update t set money = money+1000 where user='jack'");
            xaResource2.end(xid2, XAResource.TMSUCCESS);

            int ret1 = xaResource1.prepare(xid1);
            int ret2 = xaResource2.prepare(xid2);
			//两个操作都正常运行才提交事务
            if (ret1 == XAResource.XA_OK && ret2 == XAResource.XA_OK) {
                xaResource1.commit(xid1, false);
                xaResource2.commit(xid2, false);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

本博客内容，是拜读《MySQL技术内幕：SQL编程》之后，所有例子都经过自己实验，然后对书中内容进行自己的再次归纳，内容显然和书中内容差别比较大，因为是自己能理解的知识点笔录，不能理解的本博客不做记录，读者需要自行学习书籍，《MySQL技术内幕：SQL编程》一书是国人mysql大师编写的一本经典书籍，书中例子和实例都是网上博客很难搜索到的，佩服作者的高深造就，本人学习之后，虽然只能理解几成，不过也觉得受益匪浅


