## Nosql

> NoSQL：即Not-Only SQL（泛指非关系型的数据库），作为关系型数据库的补充。
>
> 作用：应对基于海量用户和海量数据前提下的数据处理问题

特征：可扩容，可伸缩;大数据量下高性能;灵活的数据模型;高可用

常见Nosql 数据库：Redis;memcache;HBase;MongoDB

<img src="..\笔记图片\电商解决方案.png" alt="电商解决方案" style="zoom: 50%;" />

## Redis基础

> Redis (REmote DIctionaryServer) 是用C 语言开发的一个开源的高性能键值对（key-value）数据库。

* 特征：

  1. 数据间没有必然的关联关系
  2. 内部采用单线程机制进行工作
  3. 高性能。官方提供测试数据，50个并发执行100000 个请求,读的速度是110000 次/s,写的速度是81000次/s。
  4. 多数据类型支持
     * 字符串类型string
     * 列表类型list
     * 散列类型hash
     * 集合类型set
     * 有序集合类型sorted_set

  5. 持久化支持。可以进行数据灾难恢复

* 应用

  * 为热点数据加速查询（主要场景），如热点商品、热点新闻、热点资讯、推广类等高访问量信息等任务队列，如秒杀、抢购、购票排队等
  * 即时信息查询，如各位排行榜、各类网站访问统计、公交到站信息、在线人数信息（聊天室、网站）、设备信号等
  * 时效性信息控制，如验证码控制、投票控制等
  * 分布式数据共享，如分布式集群架构中的session 分离
  * 消息队列
  * 分布式锁

* 核心文件：

  * redis-server.exe 服务器启动命令
  * redis-cli.exe命令行客户端
  * redis.windows.confredis核心配置文件
  * redis-benchmark.exe性能测试工具
  * redis-check-aof.exe AOF文件修复工具
  * redis-check-dump.exe RDB文件检查工具（快照持久化文件）

* 基本操作

  * 功能性命令

    * 信息添加

      set key value

    * 信息查询

      get key

  * 清除屏幕信息

    clear

  * 帮助信息查阅

    help  命令名称

    help  @组名

  * 退出指令

    quit/exit/\<ESC\>

### 数据类型

* string String

  * 存储的数据：单个数据，最简单的数据存储类型，也是最常用的数据存储类型

  * 存储数据的格式：一个存储空间保存一个数据

  * 存储内容：通常使用字符串，如果字符串以整数的形式展示，可以作为数字操作使用

  * 基本操作

    * set/get/del
    * mset/mget 
    * strlen
    * append 追加信息到原始信息后部（如果原始信息存在就追加，否则新建）

  * 扩展操作

    1. 设置数值数据增加指定范围的值解决方案

       incr key/ incrby key increment/ incrbyfloat key increment

    2. 设置数值数据减少指定范围的值

       decr key/ decrby key increment

    3. string作为数值操作
       * string在redis内部存储默认就是一个字符串，当遇到增减类操作incr，decr时会转成数值型进行计算。
       * redis所有的操作都是原子性的，采用单线程处理所有业务，命令是一个一个执行的，因此无需考虑并发带来的数据影响。
       * 按数值进行操作的数据，如果原始数据不能转成数值，或超越了redis数值上限范围，将报错。9223372036854775807（java中long型数据最大值，Long.MAX_VALUE）
       * Tips ：redis用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性此方案适用于所有数据库，且支持数据库集群

    4. 设置数据具有指定的生命周期

       setex key seconds value/psetex key milliseconds value

       Tips ：redis控制数据的生命周期，通过数据是否失效控制业务行为，适用于所有具有时效性限定控制的操作

  * string 类型数据操作的注意事项

    * 数据操作不成功的反馈与数据正常操作之间的差异

    * ①表示运行结果是否成功

      (integer) 0  →  false失败

      (integer) 1  →  true成功

    * ②表示运行结果值

      (integer) 3→  3 3个

      (integer) 1→  1 1个

      数据未获取到（nil）等同于null

    * 数据最大存储量512MB

    * 数值计算最大范围（java中的long的最大值）9223372036854775807

    * Tips: redis应用于各种结构型和非结构型高热度数据访问加速，数据库中的热点数据key命名惯例

      表名: 主键名:主键值:字段名	order​ : id : 29438 : name

* hash HashMap

  * 底层使用哈希表结构实现数据存储

  * hash存储结构优化
    * 如果field数量较少，存储结构优化为类数组结构
    * 如果field数量较多，存储结构使用HashMap结构

  * 基本操作

    * 添加修改 hset key field value
    * 获取数据 hget key field / hgetall key
    * 删除数据 hdel key field1 (field2)
    * 添加/修改多个数据 hmset key field1 value1 field2 value2 ...
    * 获取多个数据 hmget key field1 field2 ...
    * 获取哈希表中字段的数量 hlen key
    * 获取哈希表中是否存在指定的字段 hexists key field

  * 扩展操作

    * 获取哈希表中所有的字段名或字段值 hkeys key / hvals key

    * 设置指定字段的数值数据增加指定范围的值 

      hincrby key field increment / hincrbyfloat key field increment

  * hash类型数据操作的注意事项

    * hash类型下的value只能存储字符串，不允许存储其他数据类型，不存在嵌套现象。如果数据未获取到，对应的值为（nil）
    * 每个hash 可以存储2的32次方-1 个键值对
    * hash类型十分贴近对象的数据存储形式，并且可以灵活添加删除对象属性。但hash设计初衷不是为了存储大量对象而设计的，切记不可滥用，更不可以将hash作为对象列表使用
    * hgetall操作可以获取全部属性，如果内部field过多，遍历整体数据效率就很会低，有可能成为数据访问瓶颈

  * hash类型应用场景

    * 优化以用户为key的设计

      每条购物车中的商品记录保存成两条field

      * field1专用于保存购买数量

        命名格式：商品id:nums 保存数据：数值

      * field2专用于保存购物车中显示的信息，包含文字描述，图片地址，所属商家信息等

        命名格式：商品id:info 保存数据：json独立

    * hsetnx key field value

      Set the value of a hash field, only if the field does not exist

    * string存储对象（json）与hash存储对象

    * 应用于抢购，限购类、限量发放优惠卷、激活码等业务的数据存储设计

* list LinkedList

  * 数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分

  * 需要的存储结构：一个存储空间保存多个数据，且通过数据可以体现进入顺序

  * list类型：保存多个数据，底层使用双向链表存储结构实现

  * 基本操作

    * 添加/修改数据

      lpushkey value1 [value2] ......

      rpushkey value1 [value2] ......

    * 获取数据

      获取范围值（-1代表倒数第一个，-2倒数第二个）lrange key start stop 

      获取对应索引的值lindex key index

      获取对应list长度 llen key

    * 获取并移除数据

      lpop key 

      rpop key

  * 扩展操作

    * 规定时间内获取并移除数据

      blpop key1 [key2] timeout

      brpop key1[key2] timeout 

      brpoplpush source destination timeout

    * 移除指定数据

      lrem key count value

  * 注意事项

    * list中保存的数据都是string类型的，数据总容量是有限的，最多2的32次方-1个元素(4294967295)。
    * list具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作，或以栈的形式进行入栈出栈操作
      * 获取全部数据操作结束索引设置为-1
    * list可以对数据进行分页操作，通常第一页的信息来自于list，第2页及更多的信息通过数据库的形式加载
    * 应用于最新消息展示

* set HashSet

  * 新的存储需求：存储大量的数据，在查询方面提供更高的效率

  * 需要的存储结构：能够保存大量的数据，高效的内部存储机制，便于查询

  * set类型：与hash存储结构完全相同，仅存储键，不存储值（nil），并且值是不允许重复的

  * 基本操作

    * 添加数据 sadd key member1 [member2]
    * 删除数据 srem key member1 [member2]
    * 获取全部数据 smembers key 
    * 获取集合数据总量 scard key
    * 判断集合中是否包含指定数据 sismember key member

  * 扩展操作

    * 随机获取集合中指定数量的数据解决方案 srandmember key [count]

    * 随机获取集合中的某个数据并将该数据移出集合 spop key [count]

    * 应用于随机推荐类信息检索，例如热点歌单推荐，热点新闻推荐，热卖旅游线路，应用APP推荐，大V推荐等

    * 求两个集合的交、并、差集

      sinter key1 [key2] 

      sunion key1 [key2] 

      sdiff key1 [key2]

    * 求两个集合的交、并、差集并存储到指定集合中

      sinter store destination key1 [key2] 

      sunion store destination key1 [key2] 

      sdiff store destination key1 [key2] 

    * 将指定数据从原始集合中移动到目标集合中

      smove source destination member  

  * 应用场景

    * redis应用于同类型数据的快速去重
    * redis应用于基于黑名单与白名单设定的服务控制

* sorted_set TreeSet

  * 新的存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式

  * 需要的存储结构：新的存储模型，可以保存可排序的数据

  * sorted_set类型：在set的存储结构基础上添加可排序字段

  * 基础操作

    * 添加数据

      zadd key score1 member1 [score2 member2]

    * 获取全部数据

      zrange key start stop [WITHSCORES]

      zrevrange key start stop [WITHSCORES]

    * 删除数据

      zrem key member [member ...]

    * 按条件获取数据

      zrangebyscore key min max [WITHSCORES] [LIMIT]

      zrevrangebyscore key max min [WITHSCORES]

    * 条件删除数据

      zremrangebyrank key start stop

      zremrangebyscore key min max

    * 注意：

      min与max用于限定搜索查询的条件

      start与stop用于限定查询范围，作用于索引，表示开始和结束索引

      offset与count用于限定查询范围，作用于查询结果，表示开始位置和数据总量

    * 获取集合数据总量

      zcard key

      zcount key min max

    * 集合交、并操作

      zinterstore destination numkeys key [key ...]

      ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]

      zunionstore destination numkeys key [key ...]

      ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]

  * 扩展操作

    * 获取数据对应的索引（排名）

      zrank key member

      zrevrank key member

    * score值获取与修改

      zscorekey member

      zincrbykey increment member

    * Tips：应用于计数器组合排序功能对应的排名

  * 注意事项

    * score保存的数据存储空间是64位，如果是整数范围是-9007199254740992~9007199254740992
    * score保存的数据也可以是一个双精度的double值，基于双精度浮点数的特征，可能会丢失精度，使用时候要慎重
    * sorted_set底层存储还是基于set结构的，因此数据不能重复，如果重复添加相同的数据，score值将被反复覆盖，保留最后一次修改的结果

  * 应用场景

    * 应用于定时任务执行顺序管理或任务过期管理
      1. 对于基于时间线限定的任务处理，将处理时间记录为score值，利用排序功能区分处理的先后顺序
      2. 记录下一个要处理的时间，当到期后处理对应任务，移除redis中的记录，并记录下一个要处理的时间
      3. 当新任务加入时，判定并更新当前下一个要处理的任务时间
      4. 为提升sorted_set的性能，通常将任务根据特征存储成若干个sorted_set。例如1小时内，1天内，周内，月内，季内，年度等，操作时逐级提升，将即将操作的若干个任务纳入到1小时内处理的队列中
      5. 获取当前系统时间time
    * 应用于即时任务/消息队列执行管理
      1. 对于带有权重的任务，优先处理权重高的任务，采用score记录权重即可多条件任务权重设定
         * 如果权重条件过多时，需要对排序score值进行处理，保障score值能够兼容2条件或者多条件，例如外贸订单优先于国内订单，总裁订单优先于员工订单，经理订单优先于员工订单
      2. 因score长度受限，需要对数据进行截断处理，尤其是时间设置为小时或分钟级即可（折算后）
      3. 先设定订单类别，后设定订单发起角色类别，整体score长度必须是统一的，不足位补0。第一排序规则首位不得是0
         * 例如外贸101，国内102，经理004，员工008。
         * 员工下的外贸单score值为101008（优先）
         * 经理下的国内单score值为102004

* 案例

  * redis应用于限时按次结算的服务控制

    人工智能领域的语义识别与自动对话将是未来服务业机器人应答呼叫体系中的重要技术，百度自研用户评价语义识别服务，免费开放给企业试用，同时训练百度自己的模型。现对试用用户的使用行为进行限速，限制每个用户每分钟最多发起10次调用1...10

  * 解决方案

    * 设计计数器，记录调用次数，用于控制业务执行次数。以用户id作为key，使用次数作为value
    * 在调用前获取次数，判断是否超过限定次数不超过次数的情况下，每次调用计数+1业务调用失败，计数-1
    * 为计数器设置生命周期为指定周期，例如1秒/分钟，自动清空周期内使用次数

  * 解决方案优化

    * 取消最大值的判定，利用incr操作超过Long.Max最大值抛出异常的形式替代每次判断是否大于最大值
    * 判断是否为nil，如果是，设置为Max-次数如果不是，计数+1业务调用失败，计数-1
    * 遇到异常即+操作超过上限，视为使用达到上限

  * redis应用于基于时间顺序的数据操作，而不关注具体时间

    使用微信的过程中，当微信接收消息后，会默认将最近接收的消息置顶，当多个好友及关注的订阅号同时发送消息时，该排序会不停的进行交替。同时还可以将重要的会话设置为置顶。一旦用户离线后，再次打开微信时，消息该按照什么样的顺序显示

  * 解决方案

    * 依赖list的数据具有顺序的特征对消息进行管理，将list结构作为栈使用
    * 对置顶与普通会话分别创建独立的list分别管理
    * 当某个list中接收到用户消息后，将消息发送方的id从list的一侧加入list（此处设定左侧）
    * 多个相同id发出的消息反复入栈会出现问题，在入栈之前无论是否具有当前id对应的消息，先删除对应id
    * 推送消息时先推送置顶会话list，再推送普通会话list，推送完成的list清除所有数据
    * 消息的数量，也就是微信用户对话数量采用计数器的思想另行记录，伴随list操作同步更新

### 通用命令

* key通用操作

  * 特征

    key是一个字符串，通过key获取redis中保存的数据

  * 基本操作

    * 获取key是否存在 exists key
    * 获取key的类型 type key
    * 删除指定key del key

  * 扩展操作

    * 为指定key设置有效期

      expire key seconds

      pexpire key milliseconds

      expireat key timestamp 

      pexpireat key milliseconds-timestamp

    * 获取key的有效时间

      ttl key 返回-2表示key不存在，返回-1表示key存在，若有有效时间返回有效时间

      pttl key

    * 切换key从时效性转换为永久性 persist key

    * 查询key pattern key

      \*匹配任意数量的任意符号

      ? 配合一个任意符号

      []匹配一个指定符号

  * 其他操作

    * 改名

      rename key newkey

      renamex key newkey（若newkey不存在才改名

    * 排序 sort

    * 其他key通用操作 help @generic

* 数据库通用操作

  * 解决key 的重复问题

    * redis为每个服务提供有16个数据库，编号从0到15
    * 每个数据库之间的数据相互独立

  * DB基本操作

    * 切换数据库 select index

    * 其他操作

      quit	ping测试连通	echo输出控制台

  * DB相关操作

    * 数据移动 move key db
    * 数据清除 dbsize flushdb flushall

### Jedis

* 基于连接池获取连接

  * JedisPool：Jedis提供的连接池技术
  * poolConfig:连接池配置对象
  * host:redis服务地址
  * port:redis服务端口号

  ```java
  public JedisPool(GenericObjectPoolConfig poolConfig, String host, int port){
  	this(poolConfig, host, port, 2000, (String)null, 0, (String)null);
  }
  ```

  * jedis.properties

    * jedis.host=localhost
    * jedis.port=6379
    * jedis.maxTotal=30
    * jedis.maxIdle=10

  * 静态代码块初始化资源

    ```java
    static{
    	//读取配置文件获得参数值
    	ResourceBundle rb = ResourceBundle.getBundle("jedis");
        host = rb.getString("jedis.host");
        port = Integer.parseInt(rb.getString("jedis.port"));
        maxTotal = Integer.parseInt(rb.getString("jedis.maxTotal"));
        maxIdle = Integer.parseInt(rb.getString("jedis.maxIdle"));
        poolConfig = new JedisPoolConfig();poolConfig.setMaxTotal(maxTotal);
        poolConfig.setMaxIdle(maxIdle);
        jedisPool = new JedisPool(poolConfig,host,port);
    }
    ```

  * 对外访问接口，提供jedis连接对象，连接从连接池获取

    ```java
    public static Jedis getJedis(){
    	Jedis jedis = jedisPool.getResource();return jedis;
    }
    ```

## Redis高级

### Redis安装

* 基于Center OS7安装Redis

  * 下载安装包wgethttp://download.redis.io/releases/redis-?.?.?.tar.gz
  * 解压tar –xvf文件名.tar.gz
  * 编译make
  * 安装make install [destdir=/目录]

* Redis基础环境设置

  * 创建软链接ln -s 原始目录名快速访问目录名

  * 创建配置文件管理目录

    mkdir conf / mkdir config

  * 创建数据文件管理目录mkdirdata

* Redis服务启动

  * 默认配置启动

    redis-serverredis-server  –-port 6379

    redis-server  –-port 6380 ......

  * 指定配置文件启动

    redis-server  redis.conf

    redis-server  redis-6379.conf

    redis-server  redis-6380.conf ......

    redis-server  conf/redis-6379.conf

    redis-server  config/redis-6380.conf ......

  * Redis客户端连接

    * 默认连接

      redis-cli

    * 连接指定服务器

      redis-cli  -h  127.0.0.1

      redis-cli  –port  6379

      redis-cli  -h  127.0.0.1  –port   6379

  * Redis服务端配置

  * 基本配置

    * daemonize yes

      以守护进程方式启动，使用本启动方式，redis将以服务的形式存在，日志将不再打印到命令窗口中

    * port  6\*\*\*

      设定当前服务启动端口号

    * dir“/自定义目录/redis/data“

      设定当前服务文件保存位置，包含日志文件、持久化文件（后面详细讲解）等

    * logfile"6\*\*\*.log“

      设定日志文件名，便于查阅

### 持久化

* 简介

  * 概念

    利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复的工作机制称为持久化

  * 意义

    防止数据的意外丢失，确保数据安全性

  * 两种方式

    * RDB将当前数据状态进行保存，快照形式，存储数据结果，存储格式简单，关注点在数据
    * AOF将数据的操作过程进行保存，日志形式，存储操作过程，存储格式复杂，关注点在数据的操作过程

* RDB

  * 启动方式save

    手动执行一次保存

  * save指令相关配置

    * dbfilename dump.rdb

      说明：设置本地数据库文件名，默认值为dump.rdb

      经验：通常设置为dump-端口号.rdb

    * dir

      说明：设置存储.rdb文件的路径

      经验：通常设置成存储空间较大的目录中，目录名称data

    * rdbcompression yes

      说明：设置存储至本地数据库时是否压缩数据，默认为yes，采用LZF 压缩

      经验：通常默认为开启状态，如果设置为no，可以节省CPU 运行时间，但会使存储的文件变大（巨大）

    * rdbchecksum yes

      说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行

      经验：通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但是存储一定的数据损坏风险

  * 启动方式bgsave

    手动启动后台保存操作，但不是立即执行

  * 工作原理

    <img src="..\笔记图片\bgsave指令工作原理.png" alt="bgsave指令工作原理" style="zoom: 50%;" />

  * bdsave相关配置

    * dbfilename dump.rdb

    * dir

    * rdbcompression yes

    * rdbchecksum yes

    * stop-writes-on-bgsave-error yes

      说明：后台存储过程中如果出现错误现象，是否停止保存操作

      经验：通常默认为开启状态
  
  * save配置
  
    * 配置
  
      save second changes 
  
    * 作用
  
      满足限定时间范围内key的变化数量达到指定数量即进行持久化
  
    * 参数
  
      second：监控时间范围
  
      changes：监控key的变化量
  
      位置在conf文件中进行配置
  
    * 范例
  
      save 900 1 
  
      save 300 10
  
      save 60 10000
  
    * 原理
  
      <img src="..\笔记图片\save配置原理.png" alt="save配置原理" style="zoom: 50%;" />
  
    * 注意
  
      save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的
  
      save配置中对于second与changes设置通常具有互补对应关系，尽量不要设置成包含性关系
  
      save配置启动后执行的是bgsave操作
  
  * rdb特殊启动形式
  
    * 全量复制
  
    * 服务器运行过程中重启
  
      debug reload
  
    * 关闭服务器时指定保存数据
  
      shutdown save
  
  * 优劣
  
    * 优势
  
      RDB是一个紧凑压缩的二进制文件，存储效率较高
  
      RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景
  
      RDB恢复数据的速度要比AOF快很多
  
      应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。
  
    * 劣势
  
      RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据
  
      bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能
  
      Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象
  
* AOF

  * 简介

    * AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程
    * AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

  * 写数据的策略

    * always(每次）每次写入操作均同步到AOF文件中，数据零误差，性能较低，不建议使用。
    * everysec（每秒）每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高，建议使用，也是默认配置在系统突然宕机的情况下丢失1秒内的数据
    * no（系统控制）由操作系统控制每次同步到AOF文件的周期，整体过程不可控

  * AOF功能开启

    * 配置 appendonly yes|no

      作用 是否开启AOF持久化功能，默认为不开启状态

    * 配置 appendfsync always|everysec|no 

      作用 AOF 写数据策略

    * 配置 appendfilename filename

      作用  AOF持久化文件名，默认文件名未appendonly.aof，建议配置为 appendonly-端口号.aof

    * 配置 dir

      作用 AOF持久化文件保存路径，与RDB持久化文件保持一致即可

  * AOF重写

    随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录。

    * 作用

      降低磁盘占用量，提高磁盘利用率

      提高持久化效率，降低持久化写时间，提高IO性能

      降低数据恢复用时，提高数据恢复效率

    * AOF重写规则

      1. 进程内已超时的数据不再写入文件
      2. 忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令如del key1、hdelkey2、sremkey3、set key4 111、set key4 222等
      3. 对同一数据的多条写命令合并为一条命令如lpushlist1 a、lpushlist1 b、lpushlist1 c 可以转化为：lpushlist1 a b c。为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素

    * AOF重写方式

      * 手动重写 

        bgrewriteaof 

      * 自动重写 

        auto-aof-rewrite-min-size size 

        auto-aof-rewrite-percentage percentage

    * 自动重写方式

      * 自动重写触发条件设置

        auto-aof-rewrite-min-size size

        auto-aof-rewrite-percentage percent

      * 自动重写触发比对参数（运行指令info Persistence获取具体信息）

        aof_current_size

        aof_base_size

      * 自动重写触发条件

        aof_current_size>=auto-aof-rewrite-min-size

        (aof_current_size-aof_base_size/aof_base_size)>=auto-aof-rewrite-percentage

    * AOF重写流程

      <img src="..\笔记图片\AOF重写流程.png" alt="AOF重写流程" style="zoom: 50%;" />

* RBD对比AOF

  <img src="..\笔记图片\RBD对比AOF.png" alt="RBD对比AOF" style="zoom: 50%;" />

  * 对数据非常敏感，建议使用默认的AOF持久化方案
    * AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出现问题时，最多丢失0-1秒内的数据。
    * 注意：由于AOF文件存储体积较大，且恢复速度较慢
  * 数据呈现阶段有效性，建议使用RDB持久化方案
    * 数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人员手工维护的），且恢复速度较快，阶段点数据恢复通常采用RDB方案
    * 注意：利用RDB实现紧凑的数据持久化会使Redis降的很低，慎重
  * 综合比对
    * RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊
    * 如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF
    * 如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB
    * 灾难恢复选用RDB
    * 双保险策略，同时开启RDB 和AOF，重启后，Redis优先使用AOF 来恢复数据，降低丢失数据的量

* 应用场景

  以下可考虑使用

  * 应用于抢购，限购类、限量发放优惠卷、激活码等业务的数据存储设计
  * 应用于具有操作先后顺序的数据控制
  * 应用于最新消息展示
  * 应用于基于黑名单与白名单设定的服务控制
  * 应用于计数器组合排序功能对应的排名

### 事务

* 简介

  redis事务就是一个命令执行的队列，将一系列预定义命令包装成一个整体（一个队列）。当执行时，一次性按照添加顺序依次执行，中间不会被打断或者干扰

  一个队列中，一次性、顺序性、排他性的执行一系列命令

* 事务的基本操作

  * 开启事务 mutli
    * 作用 设定事务的开启位置，此指令执行后，后续的所有指令均加入到事务中
  * 执行事务 exec
    * 作用 设定事务的结束位置，同时执行事务。与multi成对出现，成对使用
  * 注意：加入事务的命令暂时进入到任务队列中，并没有立即执行，只有执行exec命令才开始执行
  * 取消事务 discard
    * 作用 终止当前事务的定义，发生在multi之后，exec之前

* 事务的工作流程

<img src="..\笔记图片\事务的工作流程.png" alt="事务的工作流程" style="zoom: 50%;" />

* 事务的注意事项

  * 定义事务的过程中，命令格式输入错误

    * 处理结果

      如果定义的事务中所包含的命令存在语法错误，整体事务中所有命令均不会执行。包括那些语法正确的命令。

  * 定义事务的过程中，命令执行出现错误怎么办？

    运行错误指命令格式正确，但是无法正确的执行。例如对list进行incr操作

    * 处理结果

      能够正确运行的命令会执行，运行错误的命令不会被执行

  * 注意：已经执行完毕的命令对应的数据不会自动回滚，需要程序员自己在代码中实现回滚。

  * 手动进行事务回滚

    * 记录操作过程中被影响的数据之前的状态
      1. 单数据：string
      2. 多数据：hash、list、set、zset
    * 设置指令恢复所有的被修改的项
      1. 单数据：直接set（注意周边属性，例如时效）
      2. 多数据：修改对应值或整体克隆复制

* 锁

  * 对key 添加监视锁，在执行exec前如果key发生了变化，终止事务执行

    watch key1 [key2......]

  * 取消对所有key 的监视

    unwatch

  * 应用基于状态控制的批量任务执行

* 应用基于分布式锁对应的场景控制

  * 业务场景

    天猫双11热卖过程中，对已经售罄的货物追加补货，且补货完成。客户购买热情高涨，3秒内将所有商品购买完毕。本次补货已经将库存全部清空，如何避免最后一件商品不被多人同时购买？【超卖问题】

  * 分析

    使用watch监控一个key有没有改变已经不能解决问题，此处要监控的是具体数据

    虽然redis是单线程的，但是多个客户端对同一数据同时进行操作时，如何避免不被同时修改

  * 方案

    * 使用setnx设置一个公共锁利用 setnx lock-key value

      setnx命令的返回值特征，有值则返回设置失败，无值则返回设置成功

    * 对于返回设置成功的，拥有控制权，进行下一步的具体业务操作
    * 对于返回设置失败的，不具有控制权，排队或等待操作完毕通过del操作释放锁
    * 注意：上述解决方案是一种设计概念，依赖规范保障，具有风险性

  * 改良

    * 使用expire为锁key添加时间限定，到时不释放，放弃锁

      expire lock-key second 

      pexpire lock-key milliseconds

    * 由于操作通常都是微秒或毫秒级，因此该锁定时间不宜设置过大。具体时间需要业务测试后确认。

      例如：持有锁的操作最长执行时间127ms，最短执行时间7ms。

      测试百万次最长执行时间对应命令的最大耗时，测试百万次网络延迟平均耗时

      锁时间设定推荐：最大耗时*120%+平均网络延迟*110%

      如果业务最大耗时<<网络平均延迟，通常为2个数量级，取其中单个耗时较长即可

### Redis删除策略

* 过期数据
  * Redis是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过TTL指令获取其状态
    * XX：具有时效性的数据
    * -1：永久有效的数据
    * -2：已经过期的数据或被删除的数据或未定义的数据Redis中的数据特征

* 数据删除策略

  1. 定时删除

     * 创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作
     * 优点：节约内存，到时就删除，快速释放掉不必要的内存占用
     * 缺点：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量
     * 总结：用处理器性能换取存储空间（拿时间换空间）

  2. 惰性删除

     * 数据到达过期时间，不做处理。等下次访问该数据时如果未过期，返回数据，发现已过期，删除，返回不存在
     * 优点：节约CPU性能，发现必须删除的时候才删除
     * 缺点：内存压力很大，出现长期占用内存的数据
     * 总结：用存储空间换取处理器性能 expireIfNeeded()（拿时间换空间）

  3. 定期删除

     周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度

     * Redis启动服务器初始化时，读取配置server.hz的值，默认为10

     * 每秒钟执行server.hz次serverCron()->databasesCron()->activeExpireCycle()

     * activeExpireCycle()对每个expires[*]逐一进行检测，每次执行250ms/server.hz*

     * 对某个expires[\*]检测时，随机挑选W个key检测

       1. 如果key超时，删除key
       2. 如果一轮中删除的key的数量>W*25%，循环该过程*
       3. 如果一轮中删除的key的数量≤W\*25%，检查下一个expires[\*]，0-15循环
       4. W取值=ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP属性值

     * 参数current_db用于记录activeExpireCycle()进入哪个expires[*]执行

     * 如果activeExpireCycle()执行时间到期，下次从current_db继续向下执行

     * 特点：

       CPU性能占用设置有峰值，检测频度可自定义设置

       内存压力不是很大，长期占用内存的冷数据会被持续清理

  4. 对比

     * 定时删除

       节约内存，无占用 不分时段占用CPU资源，频度高 拿时间换空间

     * 惰性删除

       内存占用严重 延时执行，CPU利用率高 拿空间换时间

     * 定期删除

       内存定期随机清理 每秒花费固定的CPU资源维护内存 随机抽查，重点抽查

* 逐出算法

  * 新数据进入检测

    * Redis使用内存存储数据，在执行每一个命令前，会调用freeMemoryIfNeeded()检测内存是否充足。如果内存不满足新加入数据的最低存储要求，redis要临时删除一些数据为当前指令清理存储空间。清理数据的策略称为逐出算法。
    * 注意：逐出数据的过程不是100%能够清理出足够的可使用的内存空间，如果不成功则反复执行。当对所有数据尝试完毕后，如果不能达到内存清理的要求，将出现错误信息

  * 影响数据逐出的相关配置

    * 最大可使用内存 maxmemory

      占用物理内存的比例，默认值为0，表示不限制。生产环境中根据需求设定，通常设置在50%以上。

    * 每次选取待删除数据的个数 maxmemory-samples

      选取数据时并不会全库扫描，导致严重的性能消耗，降低读写性能。因此采用随机获取数据的方式作为待检测删除数据

    * 删除策略 maxmemory-policy

      达到最大内存后的，对被挑选出来的数据进行删除的策略

  * 影响数据逐出的相关配置

    * 检测易失数据（可能会过期的数据集server.db[i].expires ）

      ①volatile-lru：挑选最近最少使用的数据淘汰

      ②volatile-lfu：挑选最近使用次数最少的数据淘汰

      ③volatile-ttl：挑选将要过期的数据淘汰

      ④volatile-random：任意选择数据淘汰

    * 检测全库数据（所有数据集server.db[i].dict）

      ⑤allkeys-lru：挑选最近最少使用的数据淘汰

      ⑥allkeys-lfu：挑选最近使用次数最少的数据淘汰

      ⑦allkeys-random：任意选择数据淘汰

    * 放弃数据驱逐

      ⑧no-enviction（驱逐）：禁止驱逐数据（redis4.0中默认策略），会引发错误OOM（Out Of Memory）

  * 数据逐出策略配置依据

    使用INFO命令输出监控信息，查询缓存hit 和miss 的次数，根据业务需求调优Redis配置

### Redis核心配置

* 服务器基础配置

  * 服务器端设定

    * 设置服务器以守护进程的方式运行

      daemonize yes|no

    * 绑定主机地址

      bind 127.0.0.1

    * 设置服务器端口号

      port 6379

    * 设置数据库数量

      databases 16

  * 日志配置

    * 设置服务器以指定日志记录级别

      loglevel debug|verbose|notice|warning

    * 日志记录文件名

      logfile 端口号.log

    * 注意：日志级别开发期设置为verbose即可，生产环境中配置为notice，简化日志输出量，降低写日志IO的频度

  * 客户端配置

    * 设置同一时间最大客户端连接数，默认无限制。当客户端连接到达上限，Redis会关闭新的连接

      maxclients 0（设置为0表示不作限制

    * 客户端闲置等待最大时长，达到最大值后关闭连接。如需关闭该功能，设置为0

      timeout 300

  * 多服务器快捷配置

    * 导入并加载指定配置文件信息，用于快速创建redis公共配置较多的redis实例配置文件，便于维护include/path/server-端口号.conf

### 高级数据类型

* Bitmaps

  * 基础操作

    * 获取指定key对应偏移量上的bit值

      getbitkey offset

    * 设置指定key对应偏移量上的bit值，value只能是1或0

      setbit key offset value

  * 扩展操作

    * 业务场景

      1. 统计每天某一部电影是否被点播
      2. 统计每天有多少部电影被点播
      3. 统计每周/月/年有多少部电影被点播
      4. 统计年度哪部电影没有被点播

    * 对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中

      bitop op destKey key1 [key2...]

      and 交 or 并 not 非 xo 异或

    * 统计指定key中1的数量

      bitcount key [start end]

  * Tips ：redis应用于信息状态统计

* Hyperloglog

  * 统计独立UV

    * 原始方案：set 存储每个用户的id（字符串）
    * 改进方案：Bitmaps 存储每个用户状态（bit）
    * 全新的方案：Hyperloglog

  * 基数

    * 基数是数据集去重后元素个数
    * HyperLogLog是用来做基数统计的，运用了LogLog的算法

  * 基本操作

    * 添加数据

      pfadd key element [element ...]

    * 统计数据

      pfcount key [key ...]

    * 合并数据

      pfmerge destkey sourcekey [sourcekey...]

  * 相关说明

    * 用于进行基数统计，不是集合，不保存数据，只记录数量而不是具体数据
    * 核心是基数估算算法，最终数值存在一定误差
    * 误差范围：基数估计的结果是一个带有0.81% 标准错误的近似值
    * 耗空间极小，每个hyperloglogkey占用了12K的内存用于标记基数
    * pfadd命令不是一次性分配12K内存使用，会随着基数的增加内存逐渐增大
    * Pfmerge命令合并后占用的存储空间为12K，无论合并之前数据量多少

  * redis应用于独立信息统计

* GEO

  * redis应用于地理位置计算

  * 基本操作

    * 添加坐标点

      geoaddkey longitude latitude member [longitude latitude member ...]

    * 获取坐标点

      geoposkey member [member ...]

    * 计算坐标点距离
  
      geodistkey member1 member2 [unit]
      
    * 添加坐标点
    
      georadiuskey longitude latitude radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
    
    * 获取坐标点
    
      georadiusbymemberkey member radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
    
    * 计算经纬度
    
      geohashkey member [member ...]

## Redis集群

### 主从复制

* 简介

  * 多台服务器连接

    * 提供数据方：master主服务器，主节点，主库主客户端
    * 接收数据方：slave从服务器，从节点，从库从客户端
    * 需要解决的问题：数据同步
    * 核心工作：master的数据复制到slave中

  * 概念

    * 主从复制即将master中的数据即时、有效的复制到slave中

  * 特征：一个master可以拥有多个slave，一个slave只对应一个master

  * 职责：

    * master：

      写数据；执行写操作时，将出现变化的数据自动同步到slave；读数据（可忽略）；

    * slave:

      读数据;写数据（禁止）

  * 作用

    * 读写分离：master写、slave读，提高服务器的读写负载能力
    * 负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量
    * 故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复
    * 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式
    * 高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案

* 工作流程

  1. 建立连接阶段

     1. 建立slave到master的连接，使master能够识别slave，并保存slave端口号
        * 设置master的地址和端口，保存master信息
        * 建立socket连接
        * 发送ping命令（定时器任务）
        * 身份验证
        * 发送slave端口信息至此，主从连接成功！

     <img src="..\笔记图片\建立连接阶段工作流程.png" alt="建立连接阶段工作流程" style="zoom: 50%;" />

     2. 状态：
        * slave：保存master的地址与端口
        * master：保存slave的端口
        * 总体：之间创建了连接的socket

     3. 主从连接

        * 方式一：客户端发送命令

          slaveof <masterip> <masterport>

        * 方式二：启动服务器参数

          redis-server -slaveof <masterip> <masterport>

        * 方式三：服务器配置

          slaveof <masterip> <masterport>

        * slave系统信息

          master_link_down_since_seconds

          masterhost

          masterport

        * master系统信息

          slave_listening_port(多个)

     4. 主从断开连接
        * 客户端发送命令slaveof no one
        * 说明：slave断开连接后，不会删除已有数据，只是不再接受master发送的数据

     5. 授权访问

        * master客户端发送命令设置密码

          requirepass <password>

        * master配置文件设置密码

          config set requirepass <password>

          config get requirepass 

        * slave客户端发送命令设置密码

          redis-server–a <password>

        * slave配置文件设置密码

          masterauth <password>

        * slave启动服务器设置密码

          auth <password>

  2. 数据同步阶段工作流程

     在slave初次连接master后，复制master中的所有数据到slave；将slave的数据库状态更新成master当前的数据库状态

     <img src="..\笔记图片\数据同步阶段工作流程.png" alt="数据同步阶段工作流程" style="zoom: 50%;" />

     * 请求同步数据
     * 创建RDB同步数据
     * 恢复RDB同步数据
     * 请求部分同步数据
     * 恢复部分同步数据

     状态：

     * slave：具有master端全部数据，包含RDB过程接收的数据
     * master：保存slave当前数据同步的位置
     * 总体：之间完成了数据克隆

     说明

     * master

       1. 如果master数据量巨大，数据同步阶段应避开流量高峰期，避免造成master阻塞，影响业务正常执行

       2. 复制缓冲区大小设定不合理，会导致数据溢出。如进行全量复制周期太长，进行部分复制时发现数据已经存在丢失的情况，必须进行第二次全量复制，致使slave陷入死循环状态。

          默认 repl-backlog-size 1mb

       3. master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下30%-50%的内存用于执行bgsave命令和创建复制缓冲区

     * slave

       1. 为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的对外服务

          slave-serve-stale-data yes|no

       2. 数据同步阶段，master发送给slave信息可以理解master是slave的一个客户端，主动向slave发送命令

       3. 多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果master带宽不足，因此数据同步需要根据业务需求，适量错峰

       4. slave过多时，建议调整拓扑结构，由一主多从结构变为树状结构，中间的节点既是master，也是slave。注意使用树状结构时，由于层级深度，导致深度越高的slave与最顶层master间数据同步延迟较大，数据一致性变差，应谨慎选择

  3. 命令传播阶段

     当master数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的状态，同步的动作称为命令传播

     master将接收到的数据变更命令发送给slave，slave接收命令后执行命令

     <img src="..\笔记图片\命令传播阶段工作流程.png" alt="命令传播阶段工作流程" style="zoom: 50%;" />

     命令传播阶段出现了断网现象

     * 网络闪断闪连 忽略
     * 短时间网络中断 部分复制
     * 长时间网络中断 全量复制

     部分复制的三个核心要素

     * 服务器的运行id（run id）

       概念：服务器运行ID是每一台服务器每次运行的身份识别码，一台服务器多次运行可以生成多个运行id

       组成：运行id由40位字符组成，是一个随机的十六进制字符例如：fdc9ff13b9bbaab28db42b3d50f852bb5e3fcdce

       作用：运行id被用于在服务器间进行传输，识别身份如果想两次操作均对同一台服务器进行，必须每次操作携带对应的运行id，用于对方识别

       实现方式：运行id在每台服务器启动时自动生成的，master在首次连接slave时，会将自己的运行ID发送给slave，slave保存此ID，通过info Server命令，可以查看节点的runid

     * 主服务器的复制积压缓冲区

       概念：复制缓冲区，又名复制积压缓冲区，是一个先进先出（FIFO）的队列，用于存储服务器执行过的命令，每次传播命令，master都会将传播的命令记录下来，并存储在复制缓冲区

       * 复制缓冲区默认数据存储空间大小是1M，由于存储空间大小是固定的，当入队元素的数量大于队列长度时，最先入队的元素会被弹出，而新元素会被放入队列

       由来：每台服务器启动时，如果开启有AOF或被连接成为master节点，即创建复制缓冲区

       作用：用于保存master收到的所有指令（仅影响数据变更的指令，例如set，select）

       数据来源：当master接收到主客户端的指令时，除了将指令执行，会将该指令存储到缓冲区中

       原理：通过offset区分不同的slave当前数据传播的差异；master记录已发送的信息对应的offset；slave记录已接收的信息对应的offset

     * 主从服务器的复制偏移量

       概念：一个数字，描述复制缓冲区中的指令字节位置

       分类：

       * master复制偏移量：记录发送给所有slave的指令字节对应的位置（多个）
       * slave复制偏移量：记录slave接收master发送过来的指令字节对应的位置（一个）

       数据来源：

       * master端：发送一次记录一次
       * slave端：接收一次记录一次

       作用：同步信息，比对master与slave的差异，当slave断线后，恢复数据使用

  4. 心跳机制

     * 进入命令传播阶段候，master与slave间需要进行信息交换，使用心跳机制进行维护，实现双方连接保持在线

       master心跳：

       * 指令：PING
       * 周期：由repl-ping-slave-period决定，默认10秒
       * 作用：判断slave是否在线
       * 查询：INFO replication获取slave最后一次连接时间间隔，lag项维持在0或1视为正常

       slave心跳任务

       * 指令：REPLCONF ACK {offset}
       * 周期：1秒
       * 作用1：汇报slave自己的复制偏移量，获取最新的数据变更指令
       * 作用2：判断master是否在线

     * 注意事项

       当slave多数掉线，或延迟过高时，master为保障数据稳定性，将拒绝所有信息同步操作

       * min-slaves-to-write 2	

       * min-slaves-max-lag 8

         slave数量少于2个，或者所有slave的延迟都大于等于10秒时，强制关闭master写功能，停止数据同步

       slave数量及延迟由slave发送REPLCONF ACK命令做确认

* 常见问题

  * 伴随着系统的运行，master的数据量会越来越大，一旦master重启，runid将发生变化，会导致全部slave的全量复制操作内部优化调整方案：

    1. master内部创建master_replid变量，使用runid相同的策略生成，长度41位，并发送给所有slave
    2. 在master关闭时执行命令shutdown save，进行RDB持久化,将runid与offset保存到RDB文件中
       * repl-id repl-offset
       * 通过redis-check-rdb命令可以查看该信息
    3. master重启后加载RDB文件，恢复数据重启后，将RDB文件中保存的repl-id与repl-offset加载到内存中
       * master_repl_id= repl master_repl_offset= repl-offset
       * 通过info命令可以查看该信息作用：本机保存上次runid，重启后恢复该值，使所有slave认为还是之前的master

  * 网络环境不佳，出现网络中断，slave不提供服务

    * 问题原因

      复制缓冲区过小，断网后slave的offset越界，触发全量复制

    * 最终结果

      slave反复进行全量复制

    * 解决方案

      修改复制缓冲区大小

    * 建议设置如下：

      1. 测算从master到slave的重连平均时长second
      2. 获取master平均每秒产生写命令数据总量write_size_per_second
      3. 最优复制缓冲区空间= 2 * second * write_size_per_secondrepl-backlog-size

  * master的CPU占用过高或slave频繁断开连接

    * 问题原因

      * slave每1秒发送REPLCONF ACK命令到master
      * 当slave接到了慢查询时（keys * ，hgetall等），会大量占用CPU性能
      * master每1秒调用复制定时函数replicationCron()，比对slave发现长时间没有进行响应

    * 最终结果

      master各种资源（输出缓冲区、带宽、连接等）被严重占用

    * 解决方案

      通过设置合理的超时时间，确认是否释放

      slave slaverepl-timeout 该参数定义了超时时间的阈值（默认60秒），超过该值，释放 

  * slave与master连接断开

    * 问题原因

      * master发送ping指令频度较低
      * master设定超时时间较短
      * ping指令在网络中存在丢包

    * 解决方案

      提高ping指令发送的频度

      repl-ping-slave-period超时时间repl-time的时间至少是ping指令频度的5到10倍，否则slave很容易判定超时

  * 多个slave获取相同数据不同步

    * 问题原因

      网络信息不同步，数据发送有延迟

    * 解决方案

      * 优化主从间的网络环境，通常放置在同一个机房部署，如使用阿里云等云服务器时要注意此现象
      * 监控主从节点延迟（通过offset）判断，如果slave延迟过大，暂时屏蔽程序对该slave的数据访问slave-serve-stale-datayes|no 开启后仅响应info、slaveof等少数命令（慎用，除非对数据一致性要求很高）

### 哨兵模式

* 简介

  * 概念

    哨兵(sentinel) 是一个分布式系统，用于对主从结构中的每台服务器进行监控，当出现故障时通过投票机制选择新的master并将所有slave连接到新的master

  * 作用

    * 监控

      不断的检查master和slave是否正常运行。master存活检测、master与slave运行情况检测

    * 通知（提醒）

      当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知。

    * 自动故障转移

      断开master与slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址

    * 注意：哨兵也是一台redis服务器，只是不提供数据服务通常哨兵配置数量为单数

  * 启用哨兵模式

    * 配置一拖二的主从结构
    * 配置三个哨兵（配置相同，端口不同）参看sentinel.conf
    * 启动哨兵 redis-sentinel sentinel-端口号.conf

  * 配置哨兵

    <img src="..\笔记图片\配置哨兵.png" alt="配置哨兵" style="zoom: 50%;" />

  * 哨兵在进行主从切换过程中经历三个阶段

    <img src="..\笔记图片\监控阶段.png" alt="监控阶段" style="zoom: 40%;" />

    * 监控

      用于同步各个节点的状态信息

      * 获取各个sentinel的状态（是否在线）

      * 获取master的状态  

        master属性：runid role

        各个slave的详细信息

      * 获取所有slave的状态（根据master中的slave信息）

        slave属性：runid、role、master_host、master_port、offset、......

    * 通知

    <img src="..\笔记图片\通知阶段.png" alt="通知阶段" style="zoom: 40%;" />

    * 故障转移

      服务器列表中挑选备选master

      * 在线的

      * 响应快的

      * 与原master断开时间短的

      * 优先原则

        优先级、offset、runid

      发送指令（sentinel ）

      * 向新的master发送slaveof no one
      * 向其他slave发送slave of 新masterIP端口 

    * 总结

      1. 监控

         同步信息

      2. 通知

         保持联通

      3. 故障转移

         发现问题

         竞选负责人

         优选新master

         新master上任，其他slave切换master，原master作为slave故障回复后连接

### 集群

* 概念

  集群就是使用网络将若干台计算机联通起来，并提供统一的管理方式，使其对外呈现单机的服务效果

* 作用

  * 分散单台服务器的访问压力，实现负载均衡
  * 分散单台服务器的存储压力，实现可扩展性
  * 降低单台服务器宕机带来的业务灾难

* 数据存储设计

  * 通过算法设计，计算出key应该保存的位置

  * 将所有的存储空间计划切割成16384份，每台主机保存一部分

    每份代表的是一个存储空间，不是一个key的保存空间

  * 将key按照计算出的结果放到对应的存储空间

* 集群内部通讯设计

  * 各个数据库相互通信，保存各个库中槽的编号数据
  * 一次命中，直接返回
  * 一次未命中，告知具体位置

* 搭建方式

  * 原生安装（单条命令）
    * 配置服务器（3主3从）
    * 建立通信（Meet）
    * 分槽（Slot）
    * 搭建主从（master-slave）
  * 工具安装（批处理）

* Cluster配置

  * 添加节点

    cluster-enabled yes|no

  * cluster配置文件名，该文件属于自动生成，仅用于快速查找文件并查询文件内容

    cluster-config-file <filename>

  * 节点服务响应超时时间，用于判定该节点是否下线或切换为从节点

    cluster-migration-barrier <count>

  * master连接的slave最小数量

    cluster-node-timeout <milliseconds>

* Cluster节点操作命令

  * 查看集群节点信息

    cluster nodes

  * 进入一个从节点redis，切换其主节点

    cluster replicate <master-id>

  * 发现一个新节点，新增主节点

    cluster meet ip:port

  * 忽略一个没有solt的节点

    cluster forget <id>

  * 手动故障转移

    cluster failover

* redis-trib命令

  * 添加节点

    redis-trib.rb add-node

  * 删除节点

    redis-trib.rb del-node

  * 重新分片

    redis-trib.rb reshard

## 企业级解决方案

### 缓存预热

* 宕机

  * 请求数量较高
  * 主从之间数据吞吐量较大，数据同步操作频度较高

* 解决方案

  * 前置准备工作：
    1. 日常例行统计数据访问记录，统计访问频度较高的热点数据
    2. 利用LRU数据删除策略，构建数据留存队列例如：storm与kafka配合
  * 准备工作：
    1. 将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据
    2. 利用分布式多服务器同时进行数据读取，提速数据加载过程
    3. 热点数据主从同时
  * 预热实施：
    1. 使用脚本程序固定触发数据预热过程
    2. 如果条件允许，使用了CDN（内容分发网络），效果会更好

* 总结

  缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据

### 缓存雪崩

* 数据库服务器崩溃
  1. 系统平稳运行过程中，忽然数据库连接量激增
  2. 应用服务器无法及时处理请求
  3. 大量408，500错误页面出现
  4. 客户反复刷新页面获取数据
  5. 数据库崩溃
  6. 应用服务器崩溃
  7. 重启应用服务器无效
  8. Redis服务器崩溃
  9. Redis集群崩溃
  10. 重启数据库后再次被瞬间流量放倒
* 问题排查
  1. 在一个较短的时间内，缓存中较多的key集中过期
  2. 此周期内请求访问过期的数据，redis未命中，redis向数据库获取数据
  3. 数据库同时接收到大量的请求无法及时处理
  4. Redis大量请求被积压，开始出现超时现象
  5. 数据库流量激增，数据库崩溃
  6. 重启后仍然面对缓存中无数据可用
  7. Redis服务器资源被严重占用，Redis服务器崩溃
  8. Redis集群呈现崩塌，集群瓦解
  9. 应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃
  10. 应用服务器，redis，数据库全部重启，效果不理想
* 分析
  * 短时间范围内大量key集中过期
* 解决方案思路
  1. 更多的页面静态化处理
  2. 构建多级缓存架构Nginx缓存+redis缓存+ehcache缓存
  3. 检测Mysql严重耗时业务进行优化对数据库的瓶颈排查：例如超时查询、耗时较高事务等
  4. 灾难预警机制监控redis服务器性能指标
     * CPU占用、CPU使用率
     * 内存容量
     * 查询平均响应时间
     * 线程数
  5. 限流、降级短时间范围内牺牲一些客户体验，限制一部分请求访问，降低应用服务器压力，待业务低速运转后再逐步放开访问
* 解决方式
  1. LRU与LFU切换
  2. 数据有效期策略调整
     * 根据业务数据有效期进行分类错峰，A类90分钟，B类80分钟，C类70分钟
     * 过期时间使用固定时间+随机值的形式，稀释集中到期的key的数量
  3. 超热数据使用永久key
  4. 定期维护（自动+人工）对即将过期数据做访问量分析，确认是否延时，配合访问量统计，做热点数据的延时
  5. 加锁慎用！

### 缓存击穿

* 数据库服务器崩溃
  1. 系统平稳运行过程中
  2. 数据库连接量瞬间激增
  3. Redis服务器无大量key过期
  4. Redis内存平稳，无波动
  5. Redis服务器CPU正常
  6. 数据库崩溃
* 问题排查
  1. Redis中某个key过期，该key访问量巨大
  2. 多个数据请求从服务器直接压到Redis后，均未命中
  3. Redis在短时间内发起了大量对数据库中同一数据的访问
* 分析
  
  * 单个高热数据key过期
* 解决方案
  1. 预先设定以电商为例，每个商家根据店铺等级，指定若干款主打商品，在购物节期间，加大此类信息key的过期时长注意：购物节不仅仅指当天，以及后续若干天，访问峰值呈现逐渐降低的趋势
  2. 现场调整监控访问量，对自然流量激增的数据延长过期时间或设置为永久性key
  3. 后台刷新数据启动定时任务，高峰期来临之前，刷新数据有效期，确保不丢失
  4. 二级缓存设置不同的失效时间，保障不会被同时淘汰就行
  5. 加锁分布式锁，防止被击穿，但是要注意也是性能瓶颈，慎重！

* 总结

  缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中redis后，发起了大量对同一数据的数据库访问，导致对数据库服务器造成压力。应对策略应该在业务数据分析与预防方面进行，配合运行监控测试与即时调整策略，毕竟单个key的过期监控难度较高，配合雪崩处理策略即可

### 缓存穿透

* 现象

  1. 系统平稳运行过程中
  2. 应用服务器流量随时间增量较大
  3. Redis服务器命中率随时间逐步降低
  4. Redis内存平稳，内存无压力
  5. Redis服务器CPU占用激增
  6. 数据库服务器压力激增
  7. 数据库崩溃

* 问题排查

  1. Redis中大面积出现未命中
  2. 出现非正常URL访问

* 分析

  * 获取的数据在数据库中也不存在，数据库查询未得到对应数据
  * Redis获取到null数据未进行持久化，直接返回
  * 下次此类数据到达重复上述过程
  * 出现黑客攻击服务器

* 解决方案

  1. 缓存null对查询结果为null的数据进行缓存（长期使用，定期清理），设定短时限，例如30-60秒，最高5分钟
  2. 白名单策略
     * 提前预热各种分类数据id对应的bitmaps，id作为bitmaps的offset，相当于设置了数据白名单。当加载正常数据时，放行，加载异常数据时直接拦截（效率偏低）
     * 使用布隆过滤器（有关布隆过滤器的命中问题对当前状况可以忽略）
  3. 实施监控实时监控redis命中率（业务正常范围时，通常会有一个波动值）与null数据的占比
     * 非活动时段波动：通常检测3-5倍，超过5倍纳入重点排查对象
     * 活动时段波动：通常检测10-50倍，超过50倍纳入重点排查对象根据倍数不同，启动不同的排查流程。然后使用黑名单进行防控（运营）
  4. key加密问题出现后，临时启动防灾业务key，对key进行业务层传输加密服务，设定校验程序，过来的key校验例如每天随机分配60个加密串，挑选2到3个，混淆到页面数据id中，发现访问key不满足规则，驳回数据访问

* 总结

  缓存击穿访问了不存在的数据，跳过了合法数据的redis数据缓存阶段，每次访问数据库，导致对数据库服务器造成压力。通常此类数据的出现量是一个较低的值，当出现此类情况以毒攻毒，并及时报警。应对策略应该在临时预案防范方面多做文章。无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除。

### 性能指标监控

* 性能指标：Performance
  * latency：Redis响应一个请求的时间
  * instantaneous_ops_per_sec：平均每秒处理请求总数
  * hit rate(calculated)：缓存命中率
* 内存指标：Memory
  * used_memory：Redis分配器分配的内存量，也就是实际存储数据的内存总量
  * mem_fragmentation_ratio：used_memory_rss /used_memory比值，表示内存碎片率
  * evicted_keys：由于maxmemory限制，而被回收内存的key的总数
  * blocked_clients：由于阻塞调用(BLPOP、BRPOP、BRPOPLPUSH)而等待的客户端的数量
* 基本活动指标：Basic activity
  * connected_clients：客户端连接数
  * connected_slaves：Slave数量
  * master_last_io_seconds_ago：最近一次主从交互之后的秒数
  * keyspace：数据库中的key值总数
* 持久性指标：Persistence
  * rdb_last_save_time：最后一次持久化保存到磁盘的Unix时间戳
  * rdb_changes_since_last_save：自最后一次持久化以来数据库的更改数
* 错误指标：Error
  * rejected_connections：由于maxclients限制而拒绝的连接数量
  * keyspace_misses：keyspace未命中次数
  * master_link_down_since_seconds：主从断开的持续时间

### 监控方式

* 工具

  * Cloud Insight Redis
  * Prometheus
  * Redis-stat
  * Redis-faina
  * RedisLive
  * zabbix

* 命令

  * benchmark
  * rediscli
  * monitor
  * slowlogs

* benchmark

  * 命令
    * redis-benchmark [-h ] [-p ] [-c ] [-n <requests]> [-k ]
  * 范例1
    * redis-benchmark
    * 说明：50个连接，10000次请求对应的性能
  * 范例2
    * redis-benchmark -c 100 -n 5000
    * 说明：100个连接，5000次请求对应的性能

  <img src="..\笔记图片\benchmark命令.png" alt="benchmark命令" style="zoom: 50%;" />

* moniter

  * 命令打印服务器调试信息

* slowlog

  * 命令

    slowlog[operator]

    * get ：获取慢查询日志
    * len：获取慢查询日志条目数
    * reset ：重置慢查询日志

  * 相关配置

    * slowlog-log-slower-than 1000 #设置慢查询的时间下线，单位：ms
    * slowlog-max-len 100          #设置慢查询命令对应的日志显示长度，单位：命令数