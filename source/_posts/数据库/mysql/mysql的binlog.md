---
title: mysql的binlog
comments: true
categories:
  - 数据库
  - mysql
abbrlink: 60550
tags:
  - 数据恢复
date: 2019-06-09 14:43:12
---
活用mysql的binlog进行数据恢复
<https://blog.csdn.net/a7442358/article/details/47355515>
    在日常操作mysql的过程中可能会遇到因为操作失误导致数据丢失，由于操作之前没有进行备份，而最近备份的文件时间又早，很可能导致备份之后到现在这段时间数据的丢失，那么如何应对这种突发状况？其实mysql已经给我们提供了应对这种情况的功能，只不过这项功能默认没有开启，平时又用不到，因此没有对它进行了解，下面我们就来认识一下它吧。
    什么是binlog？
    binlog，也称为二进制日志，记录对数据发生或潜在发生更改的SQL语句，并以二进制的形式保存在磁盘中，可以用来查看数据库的变更历史（具体的时间点所有的SQL操作）、数据库增量备份和恢复（增量备份和基于时间点的恢复）、Mysql的复制（主主数据库的复制、主从数据库的复制）。
    如何开启binlog？
    首先我们可以进入mysql输入命令
[sql] view plain copy

1. show variables like '%bin%'  
我们可以通过这个命令来查询关于binlog相关的设置，其中有一个log_bin选项，如果为off，那么证明我们的binlog没有开启，如果为on证明我们的binlog已经开启，开启binlog的方法很简单，只需要打开mysql的配置文件my.ini（也可能是my.cnf），找到log-bin，去掉前面的#号，如果没有该选项，则可以手动添加。
[sql] view plain copy
1. log-bin=mysql-bin  
其中mysql-bin就是日志文件的名称了，日志文件的名称和路径都可以自定义，如果不配置路径和名称，那么该文件会出现在mysql/data目录下，名称为mysql-bin.xxxxxx。

    添加完成后重启mysql，我们就会在mysql/data目录下找到binlog日志文件了，首次使用binlog的时候会出现两个文件，一个是mysql-bin.000001，一个是mysql-bin.index，其中，000001结尾的文件就是我们需要的日志文件，它包含了我们数据库的所有增，改，删操作（查询操作不做记录），以index结尾的文件是索引文件，包含了所有的以000xxx结尾的日志文件。
    开启binlog后mysql会自动为了记录以后增，改，删操作，关于备份操作无需我们手动操作，我们只要在需要恢复数据的时候查找对应的数据即可，由于binlog存储的格式为二进制，因此我们无法直接使用，需要借助mysql提供的工具mysqlbinlog（mysqlbinlog在mysql安装目录下的bin目录下）。初次接触一个命令不知道如何使用的时候，我们可以通过帮助命令查看它如何使用
[sql] view plain copy
1. mysqlbinlog --help  
通过该命令我们可以了解到mysqlbinlog可以附带哪些参数，不同的参数对应什么状况。
   如何使用mysqlbinlog查询操作记录？
   1.读取所有数据库的操作
[sql] view plain copy
1. mysqlbinlog ../data/mysql-bin.000001  
通过该命令，我们可以看到该日志文件中记录的所有数据库的增，该，删操作。
    2.查询指定数据库的操作
[sql] view plain copy
1. mysqlbinlog --database=test ../data/mysql-bin.000001  
通过该命令，我们可以查询数据库名称为test的增，该，删操作
    3.查询指定位置的操作
    binlog每次进行记录的时候都会为其标注一个position，用于标识该操作所在的位置，与之相关的参数为--start-position（开始位置）和--stop-position（结束位置），我们可以通过position进行指定操作的查询。
     如何获取position的值？

只需要在mysql中使用show binlog events即可，每一行都记录了一条操作，其中Pos就是该操作的start-position，End_log_pos就是stop-position。我们如果需要查询上述图片中的操作，可以使用以下语句：
[sql] view plain copy

1. mysqlbinlog --start-position=4 --stop-position=98 ../data/mysql-bin.000001  
    4.查询指定时间的操作
    除了有位置标识外，binlog还有时间标识，参数为--start-datetime（开始时间）和--stop-datetime（结束时间），如果想要查询某个时间段的操作，可以使用该参数。
[sql] view plain copy
1. mysqlbinlog --start-datetime="2015-08-08 10:00:00" --stop-datetime="2015-08-08 12:00:00" ../data/mysql-bin.000001  
    常用的查询操作也就这么多了，查询操作不是我们的目的，恢复记录才是我们的目的，一切的查询都是为了恢复。
     如何恢复删除记录？
    如果你对于查询操作了如指掌了，那么恢复操作就简单的多了，因为恢复数据就是在查询的基础上，恢复的方法大致分为两种：
    1.直接使用mysqlbinlog进行查询带恢复
[sql] view plain copy
1. mysqlbinlog --start-position=4 --stop-position=98 ../data/mysql-bin.000001 | mysql -u root -p  
上述命令跟查询操作不同的地方在于尾部添加了“| mysql -u root -p”，该命令是用于连接数据库，整条命令连接起来就是恢复开始位置为4，结束位置为98的所有操作（不限于单行记录的恢复，如果想要用于连续的多行操作，只需要把最后的结束位置设置为最后一个需要进行恢复的行的End_log_pos即可）
    2.先导出查询记录，然后通过mysql source操作进行数据恢复
[sql] view plain copy
1. mysqlbinlog --start-position=4 --stop-position=98 ../data/mysql-bin.000001 > test.sql  
[sql] view plain copy
1. mysql>source test.sql  
    恢复数据实例
    首先开启binlog，然后新建一个logtest数据库，然后新建一个test数据表，插入几条测试数据，然后删除，此时我们使用binlog来恢复数据
[sql] view plain copy
1. mysqlbinlog --database=logtest --start-position=98 --stop-position=664 ../data/mysql-bin.000001 | mysql -u root -p  
数值98是对数据库logtest的首次操作的Pos，数值664是对数据库logtest删除前的最后一次操作的End_log_pos，binlog的操作记录都在mysql-bin.000001中存储着，我们可以通过上述命令来恢复已经被删除的数据库logtest，从下图中我们可以看到1070-1636重复了98-664的动作，重新建立数据库，数据表和插入数据，我们再来打开数据库，发现被删除的数据库logtest，数据表test以及插入test的数据都已经被找回来了。

    当然使用binlog恢复的时候一定要注意几个问题：
    问题一：为什么使用mysqlbinlog --database=logtest ../data/mysql-bin.000001 | mysql -u root -p恢复数据不管用？
    其实并非该命令出错，而是日志中同时记录了最后一个drop操作，在没有选在position和datetime的情况下，binlog会读取有关该数据库的所有记录，从create到drop，使用该命令的情况下会重新把create到drop的语句再执行一遍，而在我们看来跟什么都没有做差不多，但是你再次使用mysqlbinlog查看该数据库的相关操作记录的时候就会发现，从create到drop的操作出现了两次，证明该命令的确被执行过，不过由于最后有drop语句，因此被恢复的数据库信息又被删除，回到没有恢复的状态。因此该命令不适用于数据的恢复，想要恢复数据，还要根据情况选择指定的时间或者位置。
    问题二：为什么使用position参数的时候数据恢复不完全？
     选择position的时候也是非常有讲究的，就拿上图来说，如果执行命令语句的时候，只是执行到492
[sql] view plain copy
1. mysqlbinlog --database=logtest --start-position=98 --stop-position=492 ../data/mysql-bin.000001 | mysql -u root -p  
那么进行数据恢复的时候只能恢复logtest数据库和test数据表，而不能恢复数据表中的数据，Pos是一条记录的起始位置，End_log_pos是一条记录的结束 ，如果想要恢复某连续行的数据的时候，不要忘了把最后一行需要恢复的数据的End_log_pos作为stop-position，而不是把Pos作为stop-position。
注意事项
    1.每当重启mysql的时候，都会自动生成一个新的binlog文件，恢复数据的时候首先确定需要恢复的数据在哪个日志文件中，然后查找对应binlog文件进行数据恢复。
    2.binlog分别记录了一个操作的起始位置Pos和结束位置End_log_pos，当起始位置和终止位置都选择正确的时候，恢复的数据才会正确，尤其是当进行连续的多行记录进行恢复的时候，对于stop-position的选择一定要注意，最后一行的End_log_pos才是我们需要的。
    3.使用show binlog events的时候默认指定的是第一个二进制文件，如果想要查看其它的二进制文件，可以使用show binlog events in 'logname'，其中logname是个字符串，不要忘记带上引号，否则会出错。
这篇文章很好,模拟了一个完整的过程
<https://www.cnblogs.com/martinzhang/p/3454358.html>

binlog 基本认识
    MySQL的二进制日志可以说是MySQL最重要的日志了，它记录了所有的DDL和DML(除了数据查询语句)语句，以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。

    一般来说开启二进制日志大概会有1%的性能损耗(参见MySQL官方中文手册 5.1.24版)。二进制有两个最重要的使用场景: 
    其一：MySQL Replication在Master端开启binlog，Mster把它的二进制日志传递给slaves来达到master-slave数据一致的目的。 
    其二：自然就是数据恢复了，通过使用mysqlbinlog工具来使恢复数据。
    
    二进制日志包括两类文件：二进制日志索引文件（文件名后缀为.index）用于记录所有的二进制文件，二进制日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML(除了数据查询语句)语句事件。 

一、开启binlog日志：
    vi编辑打开mysql配置文件
    # vi /usr/local/mysql/etc/my.cnf
    在[mysqld] 区块
    设置/添加 log-bin=mysql-bin  确认是打开状态(值 mysql-bin 是日志的基本名或前缀名)；

    重启mysqld服务使配置生效
    # pkill mysqld
    # /usr/local/mysql/bin/mysqld_safe --user=mysql &

二、也可登录mysql服务器，通过mysql的变量配置表，查看二进制日志是否已开启 单词：variable[ˈvɛriəbəl] 变量

    登录服务器
    # /usr/local/mysql/bin/mysql -uroot -p123456

    mysql> show variables like 'log_%'; 
    +----------------------------------------+---------------------------------------+
    | Variable_name                          | Value                                 |
    +----------------------------------------+---------------------------------------+
    | log_bin                                | ON                                    | ------> ON表示已经开启binlog日志
    | log_bin_basename                       | /usr/local/mysql/data/mysql-bin       |
    | log_bin_index                          | /usr/local/mysql/data/mysql-bin.index |
    | log_bin_trust_function_creators        | OFF                                   |
    | log_bin_use_v1_row_events              | OFF                                   |
    | log_error                              | /usr/local/mysql/data/martin.err      |
    | log_output                             | FILE                                  |
    | log_queries_not_using_indexes          | OFF                                   |
    | log_slave_updates                      | OFF                                   |
    | log_slow_admin_statements              | OFF                                   |
    | log_slow_slave_statements              | OFF                                   |
    | log_throttle_queries_not_using_indexes | 0                                     |
    | log_warnings                           | 1                                     |
    +----------------------------------------+---------------------------------------+

三、常用binlog日志操作命令
    1.查看所有binlog日志列表
      mysql> show master logs;

    2.查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值
      mysql> show master status;

    3.刷新log日志，自此刻开始产生一个新编号的binlog日志文件
      mysql> flush logs;
      注：每当mysqld服务重启时，会自动执行此命令，刷新binlog日志；在mysqldump备份数据时加 -F 选项也会刷新binlog日志；

    4.重置(清空)所有binlog日志
      mysql> reset master;

四、查看某个binlog日志内容，常用有两种方式：

    1.使用mysqlbinlog自带查看命令法：
      注: binlog是二进制文件，普通文件查看器cat more vi等都无法打开，必须使用自带的 mysqlbinlog 命令查看
          binlog日志与数据库文件在同目录中(我的环境配置安装是选择在/usr/local/mysql/data中)
      在MySQL5.5以下版本使用mysqlbinlog命令时如果报错，就加上 “--no-defaults”选项
    
      # /usr/local/mysql/bin/mysqlbinlog /usr/local/mysql/data/mysql-bin.000013
        下面截取一个片段分析：

         ...............................................................................
         # at 552
         #131128 17:50:46 server id 1  end_log_pos 665   Query   thread_id=11    exec_time=0     error_code=0 ---->执行时间:17:50:46；pos点:665
         SET TIMESTAMP=1385632246/*!*/;
         update zyyshop.stu set name='李四' where id=4              ---->执行的SQL
         /*!*/;
         # at 665
         #131128 17:50:46 server id 1  end_log_pos 692   Xid = 1454 ---->执行时间:17:50:46；pos点:692 
         ...............................................................................

         注: server id 1     数据库主机的服务号；
             end_log_pos 665 pos点
             thread_id=11    线程号


    2.上面这种办法读取出binlog日志的全文内容较多，不容易分辨查看pos点信息，这里介绍一种更为方便的查询命令：

      mysql> show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];

             选项解析：
               IN 'log_name'   指定要查询的binlog文件名(不指定就是第一个binlog文件)
               FROM pos        指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
               LIMIT [offset,] 偏移量(不指定就是0)
               row_count       查询总条数(不指定就是所有行)

             截取部分查询结果：
             *************************** 20. row ***************************
                Log_name: mysql-bin.000021  ----------------------------------------------> 查询的binlog日志文件名
                     Pos: 11197 ----------------------------------------------------------> pos起始点:
              Event_type: Query ----------------------------------------------------------> 事件类型：Query
               Server_id: 1 --------------------------------------------------------------> 标识是由哪台服务器执行的
             End_log_pos: 11308 ----------------------------------------------------------> pos结束点:11308(即：下行的pos起始点)
                    Info: use `zyyshop`; INSERT INTO `team2` VALUES (0,345,'asdf8er5') ---> 执行的sql语句
             *************************** 21. row ***************************
                Log_name: mysql-bin.000021
                     Pos: 11308 ----------------------------------------------------------> pos起始点:11308(即：上行的pos结束点)
              Event_type: Query
               Server_id: 1
             End_log_pos: 11417
                    Info: use `zyyshop`; /*!40000 ALTER TABLE `team2` ENABLE KEYS */
             *************************** 22. row ***************************
                Log_name: mysql-bin.000021
                     Pos: 11417
              Event_type: Query
               Server_id: 1
             End_log_pos: 11510
                    Info: use `zyyshop`; DROP TABLE IF EXISTS `type`

      这条语句可以将指定的binlog日志文件，分成有效事件行的方式返回，并可使用limit指定pos点的起始偏移，查询条数；
      
      A.查询第一个(最早)的binlog日志：
        mysql> show binlog events\G; 
    
      B.指定查询 mysql-bin.000021 这个文件：
        mysql> show binlog events in 'mysql-bin.000021'\G;

      C.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起：
        mysql> show binlog events in 'mysql-bin.000021' from 8224\G;

      D.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，查询10条
        mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 10\G;

      E.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，偏移2行，查询10条
        mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 2,10\G;

五、恢复binlog日志实验(zyyshop是数据库)
    1.假设现在是凌晨4:00，我的计划任务开始执行一次完整的数据库备份：

      将zyyshop数据库备份到 /root/BAK.zyyshop.sql 文件中：
      # /usr/local/mysql/bin/mysqldump -uroot -p123456 -lF --log-error=/root/myDump.err -B zyyshop > /root/BAK.zyyshop.sql
        ......

        大约过了若干分钟，备份完成了，我不用担心数据丢失了，因为我有备份了，嘎嘎~~~

      由于我使用了-F选项，当备份工作刚开始时系统会刷新log日志，产生新的binlog日志来记录备份之后的数据库“增删改”操作，查看一下：
      mysql> show master status;
      +------------------+----------+--------------+------------------+
      | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
      +------------------+----------+--------------+------------------+
      | mysql-bin.000023 |      120 |              |                  |
      +------------------+----------+--------------+------------------+
      也就是说， mysql-bin.000023 是用来记录4:00之后对数据库的所有“增删改”操作。


    2.早9:00上班了，业务的需求会对数据库进行各种“增删改”操作~~~~~~~
      @ 比如：创建一个学生表并插入、修改了数据等等：
        CREATE TABLE IF NOT EXISTS `tt` (
          `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
          `name` varchar(16) NOT NULL,
          `sex` enum('m','w') NOT NULL DEFAULT 'm',
          `age` tinyint(3) unsigned NOT NULL,
          `classid` char(6) DEFAULT NULL,
          PRIMARY KEY (`id`)
         ) ENGINE=InnoDB DEFAULT CHARSET=utf8;


      导入实验数据
      mysql> insert into zyyshop.tt(`name`,`sex`,`age`,`classid`) values('yiyi','w',20,'cls1'),('xiaoer','m',22,'cls3'),('zhangsan','w',21,'cls5'),('lisi','m',20,'cls4'),('wangwu','w',26,'cls6');


      查看数据
      mysql> select * from zyyshop.tt;
      +----+----------+-----+-----+---------+
      | id | name     | sex | age | classid |
      +----+----------+-----+-----+---------+
      |  1 | yiyi     | w   |  20 | cls1    |
      |  2 | xiaoer   | m   |  22 | cls3    |
      |  3 | zhangsan | w   |  21 | cls5    |
      |  4 | lisi     | m   |  20 | cls4    |
      |  5 | wangwu   | w   |  26 | cls6    |
      +----+----------+-----+-----+---------+


      中午时分又执行了修改数据操作
      mysql> update zyyshop.tt set name='李四' where id=4;
      mysql> update zyyshop.tt set name='小二' where id=2;

      修改后的结果：
      mysql> select * from zyyshop.tt;
      +----+----------+-----+-----+---------+
      | id | name     | sex | age | classid |
      +----+----------+-----+-----+---------+
      |  1 | yiyi     | w   |  20 | cls1    |
      |  2 | 小二     | m   |  22 | cls3    |
      |  3 | zhangsan | w   |  21 | cls5    |
      |  4 | 李四     | m   |  20 | cls4    |
      |  5 | wangwu   | w   |  26 | cls6    |
      +----+----------+-----+-----+---------+


      假设此时是下午18:00，莫名地执行了一条悲催的SQL语句，整个数据库都没了：
      mysql> drop database zyyshop;


    3.此刻杯具了，别慌！先仔细查看最后一个binlog日志，并记录下关键的pos点，到底是哪个pos点的操作导致了数据库的破坏(通常在最后几步)；
    
      备份一下最后一个binlog日志文件：
      # ll /usr/local/mysql/data | grep mysql-bin
      # cp -v /usr/local/mysql/data/mysql-bin.000023 /root/

      此时执行一次刷新日志索引操作，重新开始新的binlog日志记录文件，理论说 mysql-bin.000023 这个文件不会再有后续写入了(便于我们分析原因及查找pos点)，以后所有数据库操作都会写入到下一个日志文件；
      mysql> flush logs;
      mysql> show master status;
      

    4.读取binlog日志，分析问题
      方式一：使用mysqlbinlog读取binlog日志：
        # /usr/local/mysql/bin/mysqlbinlog  /usr/local/mysql/data/mysql-bin.000023

      方式二：登录服务器，并查看(推荐)：
        mysql> show binlog events in 'mysql-bin.000023';
        
        以下为末尾片段：
        +------------------+------+------------+-----------+-------------+------------------------------------------------------------+
        | Log_name         | Pos  | Event_type | Server_id | End_log_pos | Info                                                       |
        +------------------+------+------------+-----------+-------------+------------------------------------------------------------+
        | mysql-bin.000023 |  922 | Xid        |         1 |         953 | COMMIT /* xid=3820 */                                      |
        | mysql-bin.000023 |  953 | Query      |         1 |        1038 | BEGIN                                                      |
        | mysql-bin.000023 | 1038 | Query      |         1 |        1164 | use `zyyshop`; update zyyshop.tt set name='李四' where id=4|
        | mysql-bin.000023 | 1164 | Xid        |         1 |        1195 | COMMIT /* xid=3822 */                                      |
        | mysql-bin.000023 | 1195 | Query      |         1 |        1280 | BEGIN                                                      |
        | mysql-bin.000023 | 1280 | Query      |         1 |        1406 | use `zyyshop`; update zyyshop.tt set name='小二' where id=2|
        | mysql-bin.000023 | 1406 | Xid        |         1 |        1437 | COMMIT /* xid=3823 */                                      |
        | mysql-bin.000023 | 1437 | Query      |         1 |        1538 | drop database zyyshop                                      |
        +------------------+------+------------+-----------+-------------+------------------------------------------------------------+

        通过分析，造成数据库破坏的pos点区间是介于 1437--1538 之间，只要恢复到1437前就可。


    5.现在把凌晨备份的数据恢复：
      
      # /usr/local/mysql/bin/mysql -uroot -p123456 -v < /root/BAK.zyyshop.sql;

      注: 至此截至当日凌晨(4:00)前的备份数据都恢复了。
          但今天一整天(4:00--18:00)的数据肿么办呢？就得从前文提到的 mysql-bin.000023 新日志做文章了......


    6.从binlog日志恢复数据
      
      恢复语法格式：
      # mysqlbinlog mysql-bin.0000xx | mysql -u用户名 -p密码 数据库名

        常用选项：
          --start-position=953                   起始pos点
          --stop-position=1437                   结束pos点
          --start-datetime="2013-11-29 13:18:54" 起始时间点
          --stop-datetime="2013-11-29 13:21:53"  结束时间点
          --database=zyyshop                     指定只恢复zyyshop数据库(一台主机上往往有多个数据库，只限本地log日志)
            
        不常用选项：    
          -u --user=name              Connect to the remote server as username.连接到远程主机的用户名
          -p --password[=name]        Password to connect to remote server.连接到远程主机的密码
          -h --host=name              Get the binlog from server.从远程主机上获取binlog日志
          --read-from-remote-server   Read binary logs from a MySQL server.从某个MySQL服务器上读取binlog日志

      小结：实际是将读出的binlog日志内容，通过管道符传递给mysql命令。这些命令、文件尽量写成绝对路径；

      A.完全恢复(本例不靠谱，因为最后那条 drop database zyyshop 也在日志里，必须想办法把这条破坏语句排除掉，做部分恢复)
        # /usr/local/mysql/bin/mysqlbinlog  /usr/local/mysql/data/mysql-bin.000021 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop 

      B.指定pos结束点恢复(部分恢复)：
        @ --stop-position=953 pos结束点
        注：此pos结束点介于“导入实验数据”与更新“name='李四'”之间，这样可以恢复到更改“name='李四'”之前的“导入测试数据”
        # /usr/local/mysql/bin/mysqlbinlog --stop-position=953 --database=zyyshop /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop
      
        在另一终端登录查看结果(成功恢复了)：
        mysql> select * from zyyshop.tt;
        +----+----------+-----+-----+---------+
        | id | name     | sex | age | classid |
        +----+----------+-----+-----+---------+
        |  1 | yiyi     | w   |  20 | cls1    |
        |  2 | xiaoer   | m   |  22 | cls3    |
        |  3 | zhangsan | w   |  21 | cls5    |
        |  4 | lisi     | m   |  20 | cls4    |
        |  5 | wangwu   | w   |  26 | cls6    |
        +----+----------+-----+-----+---------+

      C.指定pso点区间恢复(部分恢复)：
        更新 name='李四' 这条数据，日志区间是Pos[1038] --> End_log_pos[1164]，按事务区间是：Pos[953] --> End_log_pos[1195]；

        更新 name='小二' 这条数据，日志区间是Pos[1280] --> End_log_pos[1406]，按事务区间是：Pos[1195] --> End_log_pos[1437]；

        c1.单独恢复 name='李四' 这步操作，可这样：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=1038 --stop-position=1164 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop

           也可以按事务区间单独恢复，如下：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=953 --stop-position=1195 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop


        c2.单独恢复 name='小二' 这步操作，可这样：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=1280 --stop-position=1406 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop
    
           也可以按事务区间单独恢复，如下：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=1195 --stop-position=1437 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop


        c3.将 name='李四'、name='小二' 多步操作一起恢复，需要按事务区间，可这样：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=953 --stop-position=1437 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop


      D.在另一终端登录查看目前结果(两名称也恢复了)：
        mysql> select * from zyyshop.tt;
        +----+----------+-----+-----+---------+
        | id | name     | sex | age | classid |
        +----+----------+-----+-----+---------+
        |  1 | yiyi     | w   |  20 | cls1    |
        |  2 | 小二     | m   |  22 | cls3    |
        |  3 | zhangsan | w   |  21 | cls5    |
        |  4 | 李四     | m   |  20 | cls4    |
        |  5 | wangwu   | w   |  26 | cls6    |
        +----+----------+-----+-----+---------+

      E.也可指定时间区间恢复(部分恢复)：除了用pos点的办法进行恢复，也可以通过指定时间区间进行恢复，按时间恢复需要用mysqlbinlog命令读取binlog日志内容，找时间节点。
        比如，我把刚恢复的tt表删除掉，再用时间区间点恢复
        mysql> drop table tt;

        @ --start-datetime="2013-11-29 13:18:54"  起始时间点
        @ --stop-datetime="2013-11-29 13:21:53"   结束时间点

        # /usr/local/mysql/bin/mysqlbinlog --start-datetime="2013-11-29 13:18:54" --stop-datetime="2013-11-29 13:21:53" --database=zyyshop /usr/local/mysql/data/mysql-bin.000021 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop

      总结：所谓恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已。

常用操作
show binlog events in "mysql-bin.000002";
mysql>flushlogs;//此时就会多一个最新的bin-log日志
mysql>showmaster status;//查看最后一个bin日志
mysql>resetmaster;//清空所有的bin-log日志
mysql>mysqlbinlog–no-defaults mysql-bin.******|more//查看bin-log日志

其他介绍
这里介绍几个mysql数据恢复的工具
mysqlbinlog_flashback
<https://blog.csdn.net/jerry____wang/article/details/56285859>
项目地址 <https://github.com/58daojia-dba/mysqlbinlog_flashback>

binlog2sql
项目地址<https://github.com/danfengcao/binlog2sql>

Recovery for mysql 这个工具有官网但是用的人好像不多,具体回去测试一下
