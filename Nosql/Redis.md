

## Nosql

> NoSQL：即Not-Only SQL（泛指非关系型的数据库），作为关系型数据库的补充。
>
> 作用：应对基于海量用户和海量数据前提下的数据处理问题

特征：可扩容，可伸缩;大数据量下高性能;灵活的数据模型;高可用

常见Nosql 数据库：Redis;memcache;HBase;MongoDB

<img src="..\笔记图片\电商解决方案.png" alt="电商解决方案" style="zoom: 50%;" />

## Redis

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

