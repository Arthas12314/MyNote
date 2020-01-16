# MySQL进阶

笔记主要来自https://www.cnblogs.com/lyjun/p/11371851.html,原课程来自Bilibili

完整的mysql优化需要很深的功底，大公司甚至有专门的DBA写上述

- mysql内核
- sql优化工程师
- mysql服务器的优化
- 各种参数常量设定
- 查询语句优化
- 主从复制
- 软硬件升级
- 容灾备份
- sql编程

### mysqL-Linux版的安装

- mysql5.5

  - 下载地址：https://dev.mysql.com/downloads/mysql/5.5.html#downloads
  - 检查当前系统是否安装过mysql：
    - 查询命令：rpm -qa|grep -i mysql
    - 删除命令：rpm -e RPM软件包名称
      - 删除自带的mysql：yum -y remove mysql-libs-5.1.73-7.el6.x86_64

- 安装mysql服务端（注意提示）：

  - rpm -ivh MySQL-server-5.5.48-1.linux2.6.i386.rpm
    - 如果报错libc.so.6：https://blog.csdn.net/xiyuliuyang/article/details/90750049
    - 如果警告key ID 5072e1f5: NOKEY：https://blog.csdn.net/Aaron960214/article/details/78451321

- 安装mysql客户端

  - rpm -ivh MySQL-client-5.5.48-1.linux2.6.i386.rpm

- 查看MySQL安装时创建的mysql用户和mysql组

  - cat /etc/passwd|grep mysql
    - cat /etc/group|grep mysql
    - mysqladmin --version

- mysql服务的启+停

  - service mysql start

  - service mysql start

    - 如果报错ERROR! The server quit without updating PID file (/var/lib/mysql/localhost.localdomain.pid).

    - 解决办法：https://www.cnblogs.com/bingco/p/8068243.html

      ```
      mysql_install_db --datadir=/var/lib/mysql
      chown mysql:mysql /var/lib/mysql -R
      ```

  - 查看mysql的进程：ps -ef|grep mysql

- mysql服务启动后，开始连接

  - 首次连接成功：mysql（不需要输入密码）
    - 给root用户设置密码：/usr/bin/mysqladmin -u root password 123456

- 自启动mysql服务

  - 设置开机自启动mysql：chkconfig mysql on
    - 查看mysql的等级：chkconfig --list | grep mysql
    - 查看不同等级代表的含义：cat /etc/inittab
    - 查看开机自动服务有哪些：ntsysv

- 修改配置文件位置

  - 版本5.5：cp /usr/share/mysql/my-huge.cnf/etc/my.cnf

    - 版本5.6：cp /usr/share/mysql/**my-default.cnf** /etc/my.cnf

- 修改字符集和数据存储路径

  - 查看字符集
    - show variables like ‘character%’;
    - show variables like ‘%char%’;
    - 由于默认的是客户端和服务器都使用的latin1，所以都是乱码
    - 修改
    - 重启mysql
    - 重新连接后，原来的库由于建立于修改字符集之前，所以中文依然是乱码，而新建表中文不是乱码

* MySQL的安装位置
  * /var/lib/mysql：mysql数据库文件的存放路径
  * /usr/share/mysql：配置文件目录
  * /usr/bin：相关命令目录
  * /etc/init.d/mysql：启停相关脚本

### mysql配置文件

- 主要配置文件
  - 二进制日志log-bin
    - 主从复制
  - 错误日志log-error
    - 默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等。
  - 查询日志log
    - 默认关闭，记录查询的sql语句，如果开启会降低mysql的整体性能，因为记录日志也是需要消耗系统资源的。
  - 数据文件
    - 两系统
      - windows：D:\devSoft\MySQLServer5.5\data目录下可以挑选很多库
      - linux
        - 看看当前系统中的全部库后再进去
        - 默认路径：/var/lib/mysql
    - frm文件：存放表结构
    - myd文件：存放表数据
    - myi文件：存放表索引
  - 如何配置
    - windows：my.ini文件
    - Linux：/etc/my.cnf文件

## mysql逻辑架构介绍

- 和其它数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上，**插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。**这种架构可以根据业务的需求和时机需要选择合适的存储引擎。

<img src="..\笔记图片\MySQL逻辑架构.png" alt="MySQL逻辑架构" style="zoom:50%;" />

* 从上到下，连接层，服务层，引擎层，存储层

  * 连接层

    最上层是一些客户端和连接服务，包含本地socket通信和大多数基于客户端/服务端工具实显得类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

  * 服务层

    第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定查询表的顺序，是否能够利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存。如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

  * 引擎层

    存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取

  * 存储层

    数据存储层，主要是将数据存储在运行于罗设备的文件系统之上，并完成于存储引擎的交互

* MySQL存储引擎

  * 查看命令

    - 如何用命令查看
      - 看你的mysql现在已提供什么存储引擎：show engines;
      - 看你的mysql当前默认的存储引擎：show variables like '%storage_engine%';

  * MyISAM和InnoDB

    |  对比项  |                          MyISAM                          |                            InnoDB                            |
    | :------: | :------------------------------------------------------: | :----------------------------------------------------------: |
    |  主外键  |                          不支持                          |                             支持                             |
    |   事务   |                          不支持                          |                             支持                             |
    |  行表锁  | 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作 |  行锁，操作时只锁某一行，不对其他行有影响，适合高并发的操作  |
    |   缓存   |                        只缓存索引                        | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
    |  表空间  |                            小                            |                              大                              |
    |  关注点  |                           性能                           |                             事务                             |
    | 默认安装 |                            是                            |                              是                              |

* 索引优化分析

  * 性能下降SQL慢
    * 执行时间长，等待时间长
      - 查询语句写的烂
      - 索引失效
        - 单值索引
        - 复合索引
      - 关联查询太多join（设计缺陷或不得已的需求）
      - 服务器调优及各个参数设置（缓冲、线程数等）

* 常见通用的Join查询

  * SQL执行顺序

    * 手写

      ```mysql
      select distinct <select_list>
      from <left_table> <join_type>
      join <right_table> on <join_condition>
      where <where_condition>
      group by <group_by_list>
      having <having_condition>
      order by <order_by_condition>
      limit <limit_number>
      ```

    * 机读

      ```mysql
      from <left_table> 
      on <join_condition>
      <join_type> join <right_table>
      where <where_condition>
      group by <group_by_list>
      having <having_condition>
      select
      distinct <select_list>
      order by <order_by_condition>
      limit <limit_number>
      ```

    * 总结

      ![SQL解析顺序](..\笔记图片\SQL解析顺序.png)

* 7种Join

<img src="..\笔记图片\SQL-Join-1.png" alt="SQL-Join" style="zoom: 50%;" />

<img src="C:\Users\hawk4\Desktop\Temp\笔记\笔记图片\SQL-Join-2.png" alt="SQL-Join" style="zoom: 67%;" />

### 索引

* 定义

  * MySQL官方定义：索引（Index）是帮助MySQL高效获取数据的数据结构。索引本质是字段与实际数据的映射

  * 排好序的快速查找数据结构

    <img src="..\笔记图片\B树结构.png" alt="SQL-Join" style="zoom: 50%;" />

  * 结论

    数据本身之外，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构就是索引

  * 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。

  * **我们平常所说的索引，如果没有特别指明，都是指B树（多路搜索树）结构组织的索引。**其中聚集索引，次要索引，覆盖索引，复合索引，前缀索引，唯一索引默认的都是使用B+树索引，统称索引。当然，除了B+树这种类型的索引之外，还有哈希索引（hash index）等。

* 优势

  - 类似大学图书馆建书目索引，**提高数据检索的效率**，降低数据库的IO成本。
  - 通过索引列对数据进行排序，**降低数据排序的成本**，降低了CPU的消耗。

* 劣势

  - 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占用空间的。
  - 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。
  - 索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或者优化查询。

* MySQL索引分类

  * 单值索引：即一个索引只包含单个列，一个表可以有多个单列索引

  * 唯一索引：索引列的值必须唯一，但允许有空值

  * 复合索引：即一个索引包含多个列

  * 基本语法

    - 创建：

      ```mysql
      create [unique] index indexname on mytable(columnname(length));
      alter mytable add [unique] index [indexname] on (columnname(length))
      ```

      - 如果是char，varchar类型，length可以小于字段实际长度；如果是blob和text类型，必须指定length。

    - 删除：drop index [indexname] on mytable;

    - 查看：show index from table_name\G

    - 使用alter命令

      ```mysql
      #该语句添加一个主键，这一意味着索引值必须是唯一的，且不能为NULL
      alter table tbl_name add primary key(column_list)
      
      #这条语句创建索引的值必须是唯一的（除NULL外，NULL可能会出现多次
      alter table tbl_name add unique index_name(column_list)
      
      #添加普通索引，索引值可出现多次
      alter table tbl_name add index index_name(column_list)
      
      #该语句指定了索引为FULLTEXT，用于全文索引
      alter table tbl_name add fulltext_index index_name(column_list)
      ```

* mysql索引结构

  - BTree索引

    - 索引原理

      <img src="..\笔记图片\B-Tree索引.png" alt="SQL-Join" style="zoom: 50%;" />
      
      1. 初始化介绍
         一颗b树，浅蓝色的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示），
         如磁盘块1包含数据项17和35，包含指针P1、P2、P3，
         P1表示小于17的磁盘块，P2表示在17和35之间的磁盘块，P3表示大于35的磁盘块。
         **真实的数据存在于叶子节点**即3、5、9、10、13、15、28、29、36、60、75、79、90、99。
         **非叶子节点不存储真实的数据，只存储指引搜索方向的数据项**，如17、35并不真实存在于数据表中。
      2. 查找过程
         如果要查找数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定磁盘块1的P2指针，内存时间因为非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3由磁盘加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，总计三次IO。
      3. 真实的情况是，3层的b+树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的，如果没有索引，每个数据项都要发生一次IO，那么总共需要百万次的IO，显然成本非常非常高。
    
  - Hash索引
  
  - full-text全文索引
  
  - R-Tree索引
  
* 哪些情况需要创建索引

  - 主键自动建立唯一索引
  - 频繁作为查询条件的字段应该创建索引
  - 查询中与其它表关联的字段，外键关系建立索引
  - 频繁更新的字段不适合创建索引，因为每次更新不单单是更新了记录，还会更新索引，加重IO负担
  - where条件里用不到的字段不创建索引
  - 单键/组合索引的选择问题，who？（在高并发下倾向创建组合索引）
  - 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
  - 查询中统计或者分组字段

* 哪些情况不需要创建索引

  - 表记录太少
  - 经常增删改的表
    - Why：提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。
  - 数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

* 性能分析

  * MySQL Query Optimizer

    1. mysql中有专门负责优化 select 语句的优化器模块，主要功能：通过计算分析系统中收集到的统计信息，为窗户端请求的query提供它认为最优的执行计划（不一定是DBA认为最优的，这部分最耗费时间）

    2. 当客户端向mysql发送一条query，命令解析器模块完成请求分类，区别出是select并转发给 mysql query optimizer时，mysql query optimizer首先会对整条query进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对query中的查询条件进行简化和调整，如去掉一些无用或显而易见的条件、结构调整等。然后分析query中的hint信息（如果有），看显示hint信息是否可以完全确定该query的执行计划。如果没有hint或hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据query进行写相应的计算分析，然后再得出最后的执行计划。

  * MySQL常见瓶颈

    - CPU：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
    - IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
    - 服务器硬件的性能瓶颈：top，free，iostat和vmstat来查看系统的性能状态

  * Explain

    - 查看执行计划

      使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。

    - 用途

      1. 表的读取顺序
      2. 数据读取操作的操作类型
      3. 哪些索引可以使用
      4. 哪些索引被实际使用
      5. 表之间的应用
      6. 每张表有多少行被优化器查询

    - 使用方式

      1. Explain+SQL语句
      2. 执行计划包含的信息

      | id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | extra |
      | ---- | ----------- | ----- | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |
      |      |             |       |      |               |      |         |      |      |       |

    * 各字段解释

      - id

        - select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序
        - 三种情况：
          1. id相同，执行顺序由上至下
          2. id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
          3. id相同不同，同时存在
        - 衍生：DERIVED

      - select_type：

        查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询

        1. SIMPLE：简单的select查询，查询中不包含子查询或者UNION。
        2. PRIMARY：查询中包含任何复杂的子部分，最外层查询则被标记为PRIMARY。
        3. SUBQUERY：在FROM列表中包含的子查询被标记为DERIVED（衍生），MySQL会递归执行这些子查询，把结果放在临时表里。
        4. DERIVED：在FROM列表中包含的子查询被标记为DERIVED（衍生）。MySQL会递归执行这些子查询，把结果放在临时表里。
        5. UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED。
        6. UNION RESULT：从UNION表中获取结果的SELECT。

      - table：显示这一行的数据是关于哪些表的。

      - type：

        1. 访问类型排序

           - **type显示的是访问类型**，是较为重要的一个指标，结果值**从最好到最坏依次是**：

             **system>const>eq_ref>ref>fulltext>ref_or_null>index_merge>unique_subquery>index_subquery>range>index>All**

        2. 显示查询使用了何种类型，从最好到最差依此是：

           system>const>eq_ref>ref>range>index>All

        3. system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，这个也可以忽略不计。

        4. const：表示通过索引一次就找到了，const用于比较primary key或则unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量。

        5. eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。

        6. ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体。

        7. range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。一般就是在你的where语句中出现了between、<、>、in等的查询。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不会扫描全部索引。

        8. index：Full Index Scan，index与All区别为index类型只遍历索引树。这通常比All快，因为索引文件通常比数据文件小。（也就是说虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）

        9. all：Full Table Scan，将遍历全表以找到匹配的行。

        10. 一般来说，**得保证查询至少达到range级别，最好能达到ref**。

      - possible_keys：显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出。**但不一定被查询实际使用**。

      - key：实际使用的索引。如果为NULL，则没有使用索引。**查询中若使用了覆盖索引，则该索引仅出现在key列表中，不会出现在possible_keys列表中。**（覆盖索引：查询的字段与建立的复合索引的个数一一吻合）

      - key_len：表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好。key_len显示的值为索引字段的最大可能长度，**并非实际使用长度**，即key_len是根据表定义计算而得，不是通过表内检索出的。

      - ref：显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。**查询中与其它表关联的字段，外键关系建立索引**。

      - rows：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。

      - Extra：包含不适合在其他列中显示但十分重要的额外信息。

        1. *Using filesort*：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作成为“文件排序”。

        2. *Using temporary*：使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。

        3. *Using index*：表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。

           覆盖索引： 

           理解方式1：SELECT的数据列只需要从索引中就能读取到，不需要读取数据行，MySQL可以利用索引返回SELECT列表中 的字段，而不必根据索引再次读取数据文件，换句话说查询列要被所建的索引覆盖 

           理解方式2：索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此他不必读取整个行。 毕竟索引叶子节点存储了他们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了，一个索引 包含了（覆盖）满足查询结果的数据就叫做覆盖索引 注意： 如果要使用覆盖索引，一定要注意SELECT列表中只取出需要的列，不可SELECT *, 因为如果所有字段一起做索引会导致索引文件过大查询性能下降

        4. Using where：表明使用了where过滤。

        5. Using join buffer：使用了连接缓存。

        6. impossible where：where子句的值总是false，不能用来获取任何元组。（查询语句中where的条件不可能被满足，恒为False）

        7. select tables optimized away：在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

        8. distinct：优化distinct操作，在找到第一匹配的元组后即停止找相同值的动作。

    * Case：
    
      <img src="..\笔记图片\MySQL索引Case.png" alt="MySQL索引Case" style="zoom: 100%;" />
    
      <img src="..\笔记图片\MySQL索引Case解释.png" alt="MySQL索引Case解释" style="zoom: 100%;" />
    

### 索引优化

* 索引分析
  
  * 单表
  
    * 建表SQL
  
      ```mysql
      create table if not exists `article`(
      `id` int(10) unsigned not null primary key auto_increment,
      `author_id` int(10) unsigned not null,
      `category_id` int(10) unsigned not null,
      `views` int(10) unsigned not null,
      `comments` int(10) unsigned not null,
      `title` varbinary(255) not null,
      `content` text not null
      );
      
      insert into `article`(`author_id`,`category_id`,`views`,`comments`,`title`,`content`)
      values(1,1,1,1,'1','1'),(2,2,2,2,'2','2'),(1,1,3,3,'3','3');
      ```
  
    * 案例
  
      ```mysql
      #查询查询 category_id 为 1 且 comments 大于 1 的情况下,views 最多的 article_id
      select id,author_id
      from article
      where category_id=1 AND comments>1
      order by views desc
      limit 1;
      ```
  
      <img src="..\笔记图片\单表查询explain结果1.png" alt="单表查询explain结果1" style="zoom: 100%;" />
  
      很显然,type 是 ALL,即最坏的情况。Extra 里还出现了 Using filesort,也是最坏的情况。优化是必须的
  
      ```mysql
      create index idx_article_ccv 
      on article(category_id,comments,views);
      ```
  
      <img src="..\笔记图片\单表查询explain结果2.png" alt="单表查询explain结果2" style="zoom: 100%;" />
  
      type 变成了 range,这是可以忍受的。但是 extra 里使用 Using filesort 仍是无法接受的。但是我们已经建立了索引,为啥没用呢?这是因为按照 BTree 索引的工作原理,先排序 category_id,如果遇到相同的 category_id 则再排序 comments,如果遇到相同的 comments 则再排序 views。当 comments 字段在联合索引里处于中间位置时,因comments > 1 条件是一个范围值(所谓 range),MySQL 无法利用索引再对后面的 views 部分进行检索,即 range 类型查询字段后面的索引无效
  
      ```mysql
       drop index idx_article_ccv on article;
       
      create index idx_article_cv 
      on article(category_id,views);
      ```
  
      <img src="..\笔记图片\单表查询explain结果3.png" alt="单表查询explain结果3" style="zoom: 100%;" />
  
      可以看到,type 变为了 ref,Extra 中的 Using filesort 也消失了,结果非常理想
  
  * 两表
  
    * 建表SQL
  
      ```mysql
      CREATE TABLE IF NOT EXISTS `class` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `card` int(10) unsigned NOT NULL,
      PRIMARY KEY (`id`)
      );
      CREATE TABLE IF NOT EXISTS `book` (
      `bookid` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `card` int(10) unsigned NOT NULL,
      PRIMARY KEY (`bookid`)
      );
      
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `class`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      insert into `book`(card ) values (floor(1+(rand()*20)));
      ```
  
      ```mysql
      #explain分析
      explain select * 
      from class 
      left join book
      on class.card=book.card;
      
      #添加索引优化1
      alter table `book` add index Y(`card`)
      #添加索引优化2
      alter table `class` add index Y(`card`)
      ```
  
      <img src="..\笔记图片\两表查询explain结果1.png" alt="两表查询explain结果1" style="zoom: 100%;" />
  
      索引优化1
  
      <img src="..\笔记图片\两表查询explain结果2.png" alt="两表查询explain结果2" style="zoom: 100%;" />
  
      索引优化2
  
      <img src="..\笔记图片\两表查询explain结果3.png" alt="两表查询explain结果3" style="zoom: 100%;" />
  
    * 结论
  
      第一次优化的type变为了ref，rows优化也比较明显
  
      左连接特性，left join 条件用于确定如何从右表搜索行，左边一定都有，所以右表是我们的关键点，一定需要建立索引
  
      而右连接特性，right join 条件用于确定如何从左表搜索行，右边一定都有，所以左表是我们的关键点，一定需要建立索引
  
  * 三表
  
    * 建表
  
      ```mysql
      create table if not exists `phone`(
      	`phoneid` int(10) unsigned not null auto_increment,
      	`card` int(10) unsigned not null,
      	primary key (`phoneid`)
      );
       
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      insert into `phone`(card ) values (floor(1+(rand()*20)));
      ```
  
      ```mysql
      #explain分析
      explain select * from class left join book on class.card=book.card left join phone on book.card = phone.card;
      
      #添加索引优化
      alter table `phone` add index Z(`card`);
      alter table `book` add index Y(`card`);
      ```
  
      <img src="..\笔记图片\三表查询explain结果1.png" alt="三表查询explain结果1" style="zoom: 100%;" />
  
      索引优化
  
      <img src="..\笔记图片\三表查询explain结果2.png" alt="三表查询explain结果2" style="zoom: 100%;" />
  
      后 2 行的 type 都是 ref 且总 rows 优化很好,效果不错
  
  * 总结
  
    * 尽可能减少Join语句中的NestedLoop的循环总次数：“永远用小结果集驱动大的结果集”。优先优化NestedLoop的内层循环；
    * 保证Join语句中被驱动表上Join条件字段已经被索引；
    * 当无法保证被驱动表的Join字段被索引且内存资源充足的前提下，不要太吝惜JoinBuffer的设置；
  
* 索引失效

  * 建表

    ```mysql
    CREATE TABLE `staffs`(
    	id int primary key auto_increment,
    	name varchar(24) not null default "" comment'姓名',
    	age int not null default 0 comment '年龄',
    	pos varchar(20) not null default ""  comment'职位',
    	add_time timestamp not null default current_timestamp comment '入职时间'
    	)charset utf8 comment '员工记录表';
     
    	
    insert into staffs(name,age,pos,add_time) values('z3',22,'manage',now());
    insert into staffs(name,age,pos,add_time) values('july',23,'dev',now());
    insert into staffs(name,age,pos,add_time) values('2000',23,'dev',now());
    select * from staffs;
    
    alter table staffs add index idx_staffs_nameAgePos(name,age,pos);
    ```

  * 索引失效案例

    * 全值匹配

      ```
      explain select * from staffs where name='July';
      explain select * from staffs where name='July' and age=23;
      explain select * from staffs where name='July' and age=23 and pos='dev';
      ```

      <img src="..\笔记图片\索引失效-全值匹配1.png" alt="索引失效-全值匹配1" style="zoom: 100%;" />

      <img src="..\笔记图片\索引失效-全值匹配2.png" alt="索引失效-全值匹配2" style="zoom: 100%;" />

      <img src="..\笔记图片\索引失效-全值匹配3.png" alt="索引失效-全值匹配3" style="zoom: 100%;" />

      ```mysql
      explain select * from staffs where age=23 and pos='dev';
      explain select * from staffs where pos='dev';
      explain select * from staffs where name='July';
      ```

      <img src="..\笔记图片\索引失效-全值匹配4.png" alt="索引失效-全值匹配4" style="zoom: 100%;" />

    * **最佳左前缀法则**：

      - 如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。（==带头大哥不能死，中间兄弟不能断==哈哈哈）

    * 不在索引列上作任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描

      ```mysql
      explain select * from staffs where name='July';
      explain select * from staffs where left(name,4)='July';
      ```

      <img src="..\笔记图片\索引失效-表达式匹配.png" alt="索引失效-表达式匹配" style="zoom: 100%;" />

    * 存储引擎不能使用索引中范围条件右边的列

      ```mysql
      explain select * from staffs where name='z4';
      explain select * from staffs where name='z4' and age=22;
      explain select * from staffs where name='z4' and age=22 and pos='manager';
      explain select * from staffs where name='z4' and age>11 and pos='manager';
      ```

      <img src="..\笔记图片\索引失效-范围条件右值匹配.png" alt="索引失效-范围条件右值匹配" style="zoom: 100%;" />

    * 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *

      ```mysql
      explain select name,age,pos from staffs where name='July' and age=25 and pos='dev';
      explain select * from staffs where name='July' and age=25 and pos='dev';
      explain select name,age,pos from staffs where name='July' and age>25 and pos='dev';
      explain select name,age,pos from staffs where name='July' and age=25;
      explain select name from staffs where name='July' and age=25;
      ```

      <img src="..\笔记图片\尽量使用覆盖索引.png" alt="索引失效-范围条件右值匹配" style="zoom: 100%;" />

    * mysql在使用不等于（！=或者<>）的时候无法使用索引会导致全表扫描

      ```mysql
      explain select * from staffs where name='July';
      explain select * from staffs where name!='July';
      explain select * from staffs where name<>'July';
      ```

      <img src="..\笔记图片\索引失效-使用不等于匹配.png" alt="索引失效-使用不等于匹配" style="zoom: 100%;" />

    * is null，is not null也无法使用索引

      ```mysql
      explain select * from staffs where name is null;
      explain select * from staffs where name is not null;
      ```

    * like以通配符开头(‘%abc…’)mysql索引失效会变成全表扫描的操作

      ```mysql
      explain select * from staffs where name like '%July%';
      explain select * from staffs where name like '%July';
      explain select * from staffs where name like 'July%';
      ```

      <img src="..\笔记图片\索引失效-使用like匹配.png" alt="索引失效-使用like匹配" style="zoom: 100%;" />

      - **like%加右边**

      - 问题：解决like ‘%字符串%’时索引不被使用的方法？

        利用覆盖索引解决两边%的优化问题。

        ```mysql
        CREATE TABLE `tb1_user`(
        	id int not null auto_increment,
        	name varchar(20)  default null,
        	age int(11) default null ,
        	email varchar(20)  default null ,
        	 primary key(`id`)
        	)engine=innodb auto_increment=1 default charset=utf8;
         
        	
        insert into tb1_user(name,age,email) values ('1aa1','21','b@163.com');
        insert into tb1_user(name,age,email) values ('2aa2','222','a@163.com');
        insert into tb1_user(name,age,email) values ('3aa3','265','c@163.com');
        insert into tb1_user(name,age,email) values ('4aa4','21','d@163.com');
        
         create index idx_nameAge on tb1_user(name,age);
        ```

        ```
        explain select name,age from tb1_user where name like '%aa%';
        explain select id from tb1_user where name like '%aa%';
        explain select name from tb1_user where name like '%aa%';
        explain select age from tb1_user where name like '%aa%';
        explain select id,name from tb1_user where name like '%aa%';
        explain select name,age from tb1_user where name like '%aa%';
        
        explain select * from tb1_user where name like '%aa%';
        explain select id,name,age,email from tb1_user where name like '%aa%';
        ```

        <img src="..\笔记图片\索引失效-使用like匹配-覆盖索引.png" alt="索引失效-使用like匹配-覆盖索引" style="zoom: 100%;" />

    * 字符串不加单引号索引失效

      ```mysql
      explain select * from staffs where name='2000';
      explain select * from staffs where name=2000;
      ```

      <img src="..\笔记图片\索引失效-字符串未加单引号.png" alt="索引失效-字符串未加单引号" style="zoom: 100%;" />

      该问题同问题3，是索引列上做了类型转换！

      - **VARCHAR类型绝对不能失去单引号!**

    * 少用or，用它来连接时会索引失效

      ```
      explain select * from staffs name='July' or name='z3';
      ```

      <img src="..\笔记图片\索引失效-使用or连接.png" alt="索引失效-使用or连接" style="zoom: 100%;" />

    * 总结

      <img src="..\笔记图片\索引失效-总结.png" alt="索引失效-总结" style="zoom: 100%;" />

* 案例讲解

  * 建表

    ```mysql
    create table test03(
    	id int primary key not null auto_increment,
    	c1 char(10),
    	c2 char(10),
    	c3 char(10),
    	c4 char(10),
    	c5 char(10)
    );
    insert into test03(c1,c2,c3,c4,c5) values ('a1','a2','a3','a4','a5');
    insert into test03(c1,c2,c3,c4,c5) values ('b1','b2','b3','b4','b5');
    insert into test03(c1,c2,c3,c4,c5) values ('c1','c2','c3','c4','c5');
    insert into test03(c1,c2,c3,c4,c5) values ('d1','d2','d3','d4','d5');
    insert into test03(c1,c2,c3,c4,c5) values ('e1','e2','e3','e4','e5');
    
    ```

#索引分析
    explain select * from test03 where c1='a1';
    explain select * from test03 where c1='a1' and c2='a2';
    explain select * from test03 where c1='a1' and c2='a2' and c3='a3';
    explain select * from test03 where c1='a1' and c2='a2' and c3='a3' and c4='a4';
    #MySQL自带优化 依然可以使用四条索引
    explain select * from test03 where c1='a1' and c2='a2' and c3='a3' and c4='a4';
    #范围后失效
    explain select * from test03 where c1='a1' and c2='a2' and c3>'a3' and c4='a4';
    
    explain select * from test03 where c1='a1' and c2='a2' and c4>'a4' and c3='a3';
    
    #c3用于排序
    explain select * from test03 where c1='a1' and c2='a2' and c4>'a4' order by c3;
    explain select * from test03 where c1='a1' and c2='a2' order by c3;
    #只用到c1 c2 using filesort
    explain select * from test03 where c1='a1' and c2='a2' order by c4;
    #只用c1一个字段索引，但是c2、c3用于排序，无filesort
    explain select * from test03 where c1='a1' and c5='a5' order by c2,c3;
    #违背索引顺序，出现了filesort
    explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;
    
    explain select * from test03 where c1='a1' and c2='a2' order by c2,c3;
    #用c1,c2两个字段索引，但是c2,c3用于排序，无filesort0
    explain select * from test03 where c1='a1' and c5='a5' order by c2,c3;
    #有常量c2时为特例，无filesort
    explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c3,c2;
    
    explain select * from test03 where c1='a1' and c4='a4' group by c2,c3;
    #using temporary;using filesort
    explain select * from test03 where c1='a1' and c4='a4' group by c3,c2;
    ```

  * 定值、范围还是排序，一般order by是给个范围
  
  * group by基本上都需要进行排序，会有临时表产生
  
  * 一般性建议
  
    - 对于单键索引，尽量选择针对当前query过滤性更好的索引
    - 在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好
    - 在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引
    - 尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的

### 查询截取分析

- 分析
  1. 观察，至少跑1天，看看生产的慢SQL情况。
  2. 开启慢查询日志，设置阈值，比如超过5秒钟的就是慢SQL，并将它抓取出来。
  3. explain+慢SQL分析
  4. show profile
  5. 运维经理 or DBA，进行SQL数据库服务器参数调优。
- 总结
  1. 慢查询的开启并捕获
  2. explain+慢SQL分析
  3. show profile查询SQL在Mysql服务器里面的执行细节和生命周期情况
  4. SQL数据库服务器的参数调优

* 查询优化
  * **永远小表驱动大表**，类似嵌套循环Nested Loop

    * 优化原则：小表驱动大表，即小的数据集驱动大的数据集。

    * 当B表的数据集必须小于A表的数据集时，用in优于exists

      ```mysql
      select * from a where id in (select id from b)
      #等价于
      for select id from b
      for select * from a where a.id=b.id
      ```

    * 当A表的数据集必须小于B表的数据集时，用exists优于in

      ```mysql
      select * from a where exists(select 1 from b where b.id=a.id)
      #等价于
      for select * from a
      for select * from b where b.id=a.id
      ```

    * 注意：A表与B表的ID字段应建立索引。

    * EXISTS

      1. SELECT … FROM table WHERE EXISTS(subquery)
      2. 该语法可以理解为：**将主查询的数据，放到子查询中做条件验证，根据验证结果（TRUE或FALSE）来决定主查询的数据结果是否得以保留。**

    * 提示

      1. EXISTS（subquery）只返回TRUE或FALSE，因此子查询中的SELECT *也可以是SELECT 1或SELECT ‘X’，官方说法是实际执行时会忽略SELECT清单，因此没有区别。

      2. EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担心效率问题，可进行实际检验以确定是否有效率问题。
      3. EXISTS子查询往往也可以用条件表达式/其他子查询或者JOIN来替代，何种最优需要具体问题具体分析。

    * 总结

      in后面为小表，exists后面为大表

      对于exists的subquery使用可以理解为，讲主查询的数据放入子查询中做条件验证，根据验证结果决定主查询的数据是否保留

  * order by关键字优化

    * ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序
      * 建表

        ```mysql
        create table tb1A(
        	#id int primary key not null auto_increment,
        	age int,
        	birth timestamp not null
         
        );
        insert into tb1A(age,birth) values (22,now());
        insert into tb1A(age,birth) values (23,now());
        insert into tb1A(age,birth) values (24,now());
        
        create index idx_A_ageBirth on tb1A(age,birth);
        ```

      * Case

        ```mysql
        explain select * from tb1A where age>20 order by age;
        explain select * from tb1A where age>20 order by age,birth;
        explain select * from tb1A where age>20 order by birth;
        ```

        <img src="..\笔记图片\orderby子句案例1.png" alt="orderby子句案例1" style="zoom: 100%;" />

        ```mysql
        explain select * from tb1A order by birth;
        explain select * from tb1a where birth>'2020-01-14 00:00:00' order by birth;
        explain select * from tb1a where birth>'2020-01-14 00:00:00' order by age;
        explain select * from tb1A order by age asc,birth desc;
        ```

        <img src="..\笔记图片\orderby子句案例2.png" alt="orderby子句案例2" style="zoom: 100%;" />

      * MySQL支持两种方式的排序

        - FileSort和Index，Index效率高。FileSort方式效率较低。
        - Using Index，它指MySQL扫描索引本身完成排序。

      * ORDER BY满足两种情况，会使用Index方式排序：

        - ORDER BY语句使用索引最左前列
        - 使用Where子句与ORDER BY子句条件列组合满足索引最左前列

    * 尽可能在索引列上完成排序操作，遵照索引建的最佳最前缀

    * 如果不在索引列上，filesort有两种算法：

      mysql就要启动双路排序和单路排序

      - 双路排序

        - MySQL4.1之前是使用双路排序，字面意思就是**两次**扫描磁盘，最终得到数据。读取行指针和order by列，对他们进行排序，然后扫描已经排好序的列表，按照列表中的值重新从列表中读取对应的数据输出。
        - 从磁盘取排序字段，在buffer进行排序，再从磁盘读取其他字段。

      - 取一批数据，要对磁盘进行了两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序

      - 单路排序

        - 从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间。

      - 结论及引申出的问题

        - 由于单路是后出的，总体而言好过双路

        - 单路存在的问题

          在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出，所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序(创建tmp文件，多路合并)，排完再取sort_buffer容量大小，再排序，从而多次IO；问题即是多次IO操作（原因：数据的总大小超过sort_buffer的容量）

    * 优化策略

      1. 增大sort_buffer_size参数的设置

      2. 增大max_length_for_sort_data参数的设置

      3. 原因

         提高ORDER BY的速度

         1. order by时select * 是一个大忌，应当只Query需要的字段

            1.1 当Query的字段大小总和小于max_length_for_sort_data而且排序字段不是TEXT|BLOB类型时，回用改进后的算法-单路排序，否则用老算法多路排序

            1.2 两种算法都有可能超出sort_buffer的容量，超出过后，会创建tmp文件进行合并排序，导致多次IO（单路算法的风险更大，所以应提高sort_buffer_size

         2. 尝试提高sort_buffer_size

         3. 尝试提高max_length_for_sort_data

            提高此参数会增加改进算法的概率，但如果设置的太高，数组总容量超出sort_buffer_size的概率就增大，明显症状是高磁盘IO活动和低CPU使用率

    * 总结

      为排序使用索引

      1. MySQL两种排序方式：文件排序或扫描有序索引排序
      2. MySQL能为排序与查询使用相同的索引

      ```mysql
      KEY a_b_c(a,b,c)
      #order by能使用索引最左前缀
      order by a
      order by a,b
      order by a,b,c
      order by a desc,b desc,c desc
      
      #如果where使用索引的最左前缀定义为常量，则order by能使用索引
      where a=const order by b,c
      where a=const and b=const order by b,c
      where a=const and b>const order by b,c
      
      #不能用索引进行排序
      #排序不一致
      order by a asc,b desc,c desc;
      #丢失a索引
      where g=const order by b,c;
      #丢失b索引
      where a=const order by c;
      #d不是索引的一部分
      where a=const order by a,d;
      #对于排序来说，多个想等条件也是范围查询
      where a in(...) order by b,c
      ```

  * GROUP BY关键字优化

    - group by实质是先排序后进行分组，遵照索引建的最佳左前缀。
    - 当无法使用索引列，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置。
    - where高于having，能写在where限定的条件就不要去having限定了。

### 慢查询日志

### 数据库锁

* MySQL锁机制

  * 锁是计算机协调多个进程并发访问某一资源的机制。

    在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如果保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素，从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂
  
* 锁的分类

  - 从对数据操作的类型（读/写）分
    - 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
    - 写锁（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。
  - 从对数据操作的粒度分
    - 表锁
    - 行锁

* 开销、加锁速度、死锁、粒度、并发性能

* 只能就具体应用的特点来说那种锁更合适

* 表锁（偏读锁）

  * 特点：偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。

  * 案例分析

    - 建表

      ```mysql
      create table mylock(
      id int not null primary key auto_increment,
      name varchar(20)
      )engine myisam;
      
      insert into mylock(name) values('a');
      insert into mylock(name) values('b');
      insert into mylock(name) values('c');
      insert into mylock(name) values('d');
      insert into mylock(name) values('e');
      
      select * from mylock;
      
      #手动添加表锁
      lock table 表名字 read(write), 表名字2 read(write)
      #查看是否加锁表
      show open tables;
      #释放锁
      unlock tables;
      ```

    - 加读锁（我们为mylock表加read锁（读阻塞写例子））

      <img src="..\笔记图片\案例-读写锁1.png" alt="案例-读写锁1" style="zoom: 100%;" />

      <img src="..\笔记图片\案例-读写锁2.png" alt="案例-读写锁2" style="zoom: 100%;" />

    - 加写锁（我们为mylock表加write锁（MyISAM存储引擎的写阻塞读例子））

      <img src="..\笔记图片\案例-读写锁3.png" alt="案例-读写锁3" style="zoom: 100%;" />

  * 案例结论

    * 对MyISAM的读操作，不会阻塞其他用户对同一表请求，但会阻塞对同一表的写请求；
    * 对MyISAM的写操作，则会阻塞其他用户对同一表的读和写操作；
    * MyISAM表的读操作和写操作之间，以及写操作之间是串行的。

    * 当一个线程获得对一个表的写锁后，只有持有锁线程可以对表进行更新操作。其他线程的读、写操作都会等待，直到锁被释放为止。

  * 表锁分析

    - 看看哪些表被加锁了：show open tables;
    - 如何分析表锁定：可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定。
      - show status like ‘table%’;
      - 这里有两个状态变量记录MySQL内部表级锁定的情况，两个变量的说明如下：
        - Table_locks_immediate：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1；
        - Table_locks_waited：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在着较严重的表级锁争用情况。
      - **此外，MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。**

* 行锁（偏写锁）

  * 特点

    * 偏向Innodb存储引擎，开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率最低，并发度也最高。
    * Innodb与MyISAM的最大不同有两点：
      - 一是支持事务（TRANSACTION）
      - 而是采用了行级锁

  * 案例分析

    ```mysql
    # 行锁分析建表
    create table test(a int(11), b varchar(16))engine=innodb;
    insert into test(a,b)values(1,'b2'),
    (3,'3'),(4,'4000'),(5,'5000'),(6,'6000'),(7,'7000'),(8,'8000'),(9,'9000'),(1,'b1');
    create index idx_test_innodb_a on test(a);
    create index idx_test_innodb_b on test(b);
    select * from test;
    # 自动提交关了，必须手动写commit才能提交
    set autocommit = 0;
    ```

    <img src="..\笔记图片\案例-偏写锁1.png" alt="案例-偏写锁1" style="zoom: 100%;" />

  * 无索引行锁升级为表锁

    - 如果在更新数据的时候出现了强制类型转换导致索引失效，使得行锁变表锁，即在操作不同行的时候，会出现阻塞的现象。

  * 间隙锁危害

    ```mysql
    #进程1
    update test set b='0629' where a>1 and a<6;
    #进程2
    insert into test values(2,'2000');
    ```

    <img src="..\笔记图片\案例-间隙锁.png" alt="案例-间隙锁" style="zoom: 100%;" />

    * 当我们用范围条件而不是相等条件索引数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP）”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。
    * 危害：
      1. 因为Query执行过程中通过范围查找的话，会锁定整个范围内所有的索引键值，即使这个键值并不存在。
      2. 间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害。

  * 面试题：常考如何锁定一行

    - select * from 表 where 某一行的条件 for update;

  * 案例结论

    - InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，InnoDB的整体性能和MyISAM相比就会有比较明显的优势了。
    - 但是，InnoDB的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让InnoDB的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

  * 行锁分析

    - 如何分析行锁定

      - 通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况

        show status like ‘innodb_row_lock%’;

        <img src="..\笔记图片\案例-行锁分析.png" alt="案例-行锁分析" style="zoom: 100%;" />

      - 对各个状态量的说明如下：

        - Innodb_row_lock_current_waits：当前正在等待锁定的数量；
        - innodb_row_lock_time：从系统启动到现在锁定总时间长度；
        - innodb_row_lock_time_avg：每次等待所花平均时间；
        - innodb_row_lock_time_max：从系统启动到现在等待最长的一次所花的时间；
        - innodb_row_lock_waits：系统启动后到现在总共等待的次数。

      - 对于这5个变量，比较重要的是

        - innodb_row_lock_time_avg（等待平均时长）
        - innodb_row_lock_waits（等待总次数）
        - innodb_row_lock_time（等待总时长）
        - 这三项尤其是当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优化计划。

### 主从复制

* 复制的基本原理
  * slave会从master读取binlog来进行数据同步

  * 三步骤+原理图

    <img src="..\笔记图片\主从复制-三步骤原理图.png" alt="主从复制-三步骤原理图" style="zoom: 100%;" />

  * 复制的基本原则
  
    - 每个slave只有一个master
    - 每个slave只能有一个唯一的服务器ID
    - 每个master可以有多个slave
  
  * 复制的问题
  
    * 延时
  
  * 一主一从常见配置
  
    * mysql版本一致且后台以服务运行
    * 主从都配置在[mysqld]结点下，都是小写
    * 主机修改my.ini配置文件
      - 【必须】主服务器唯一ID
        - server-id=1
      - 【必须】启用二进制日志
        - log-bin=自己本地的路径/mysqlbin
        - log-bin=D:/devSoft/MySQLServer5.5/data/mysqlbin
      - 【可选】启用错误日志
        - log-err=自己本地的路径/mysqlerr
        - log-err=D:/devSoft/MySQLServer5.5/data/mysqlerr
      - 【可选】根目录
        - basedir=自己本地路径
        - basedir="D:/devSoft/MySQLServer5.5/"
      - 【可选】临时目录
        - tmpdir=自己本地路径
        - tmpdir="D:/devSoft/MySQLServer5.5/"
      - 【可选】数据目录
        - datadir=自己本地路径/Data/
        - datadir="D:/devSoft/MySQLServer5.5/data"
      - read-only=0
        - 主机，读写都可以
      - 【可选】设置不要复制的数据库
        - binlog-ignore-db=mysql
      - 【可选】设置需要复制的数据库
        - binlog-do-db=需要复制的主数据库名字
  
  * 从机修改my.cnf配置文件
  
    - 【必须】从服务器唯一ID
      - server-id=2
    - 【可选】启用二进制日志
  
  * 因修改过配置文件，请主机+从机都重启后台mysql服务
  
  * 主机从机都关闭防火墙
  
    - windows手动关闭
    - 关闭虚拟机linux防火墙：service iptables stop
  
  * 在Windows主机上建立账户并授权slave
  
    - `GRANT REPLICATION SLAVE ON *.* TO 'zhangsan' @ '192.168.14.167【从机数据库IP】' IDENTIFIED BY '123456';`
  
    - flush privileges;
  
    - 查询master的状态
  
      - show master status
  
        <img src="..\笔记图片\主从复制1.png" alt="主从复制1" style="zoom: 100%;" />
  
      - 记录下File和Position的值
  
    - 执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化
  
  * 在Linux从机上配置需要复制的主机
  
    ```shell
    CHANGE MASTER TO MASTER_HOST='主机IP', MASTER_USER='zhangsan', MASTER_PASSWORD='123456', MASTER_LOG_FILE='file名字', MASTER_LOG_POS=position数字;
    ```
  
    <img src="..\笔记图片\主从复制2.png" alt="主从复制2" style="zoom: 100%;" />
  
  * 启动从服务器复制功能
  
    - start slave;
  
  * show slave status\G
  
    - 下面两个参数都是Yes，则说明主从配置成功！
  
    - Slave_IO_Running：Yes
  
    - Slave_SQL_Running：Yes
  
      <img src="..\笔记图片\主从复制3.png" alt="主从复制3" style="zoom: 100%;" />
  
  * 主机新建库、新建表、insert记录，从机复制
  
  * 如何停止从服务复制功能
  
    - stop slave;