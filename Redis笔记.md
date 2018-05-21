#### Linux安装Redis

1. 下载

```bash
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
```

2. 解压

```bash
tar -zxvf redis redis-4.0.9 /usr/local
```

3. 编译

```bash
cd redis-4.0.9
make
```

4. 安装

```bash
cd src 
make install /usr/local/redis
```

5. 将配置文件移动到redis安装目录下

```bash
#redis.conf在redis-4.0.9目录下
mv redis.conf /usr/local/redis/redis.conf
```

6. 修改redis.conf 

```bash
#启用后台模式运行，默认是前台模式运行
daemonize = no	#no改为yes
#允许其它机器远程访问
bing 127.0.0.1	#注销这一句
protected-mode yes	#yes改为no
```

7. 配置环境变量

```bash
vim /etc/profile
	export REDIS_HOME = /usr/local/redis
	export path = $REDIS_HOME/bin:
source /etc/profile	#立即生效
```

8. 启动、连接、关闭

```bash
#1.前端模式启动
redis-server
#2.后端模式启动，后面要指定配置文件
redis-server /usr/local/usr/redis/redis.conf
#3.测试，客户端连接
redis-cli
#4.强制停止服务，会导致Redis持久化数据丢失
ps -ef | grep redis #查询进程端口号
pkill 6379an
#5.安全关闭服务
redis-cli shutdown
```

9. 如需远程使用，配置防火墙

```bash
vim /etc/sysconfig/iptables
#加上规则
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6379 -j ACCEPT
```

#### Redis应用场景

- 缓存（数据查询、短链接、新闻内容、商品内容等等）。（最多使用）
- 聊天室的在线好友列表。
- 任务队列（秒杀、抢购、12306等等）。
- 应用排行榜。
- 网站访问统计。
- 数据过期处理（可以精确到毫秒）。
- 分布式集群架构中的session分离。


#### Jedis：Java使用Redis

```java
import org.junit.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Properties;

public class JedisTest {

    @Test
    public void testJedis() {
        Jedis jedis = new Jedis("192.168.0.8", 6379);
        jedis.set("addr", "华健");
        System.out.println(jedis.get("addr"));
        jedis.close();
    }

    @Test
    public void testJedisPool() {

        //0.读取配置文件
        Properties prop = new Properties();
        InputStream inputStream = 	JedisTest.class.getClassLoader().getResourceAsStream("redis.properties");
        try {
            prop.load(new InputStreamReader(inputStream, "UTF-8"));
        } catch (IOException e) {
            e.printStackTrace();
        }

        //1.获得连接池配置对象，设置配置项
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(
            		Integer.parseInt(prop.getProperty("redis.maxTotal")));
        jedisPoolConfig.setMaxIdle(
            		Integer.parseInt(prop.getProperty("redis.maxIdle")));
        jedisPoolConfig.setMinIdle(
            		Integer.parseInt(prop.getProperty("redis.minIdle")));

        //2.获得连接池
        JedisPool jedisPool = new JedisPool(jedisPoolConfig, "192.168.0.8", 6379);

        Jedis jedis = null;
        try{
            //3.获得核心对象
            jedis = jedisPool.getResource();
            //4.设置数据
            jedis.set("addr", "baoying");
            //5.获得数据
            String address = jedis.get("addr");
            System.out.println(address);
        }catch (Exception e){
            e.printStackTrace();
        } finally {
            //6.释放资源
            if(jedis != null) {
                jedis.close();
            }
            if(jedisPool != null) {
                jedisPool.close();
            }
        }

    }
}
```

#### Redis支持物种数据结构

Redis是一种高级的key-value的存储系统，其中value支持五种数据结构：

- 字符串（String）
- 哈希（hash）
- 列表（list）
- 集合（set）
- 有序集合（sortedset）

1. 字符串（String）

```bash
#赋值
set key value
#取值
get key
#先取值再赋值
getset key value
#删除
del key
#数值增减，只对数字字符串有效
incr key	#将数值+1
decr key	#将数值-1
incrby key value	#将数值+value
decrby key value	#将数值-value
#拼接字符串
appeng key value	#key存在，其后追加；key不存在，创建一个key/value
```

2. 哈希（hash）

   ​	Redis中的Hash类型可以看成具有String Key和String Value的map容器。所以该类型非常适合于存储对象的信息。如Username、Password和Age等。如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间。每一个Hash可以存储4294967295个键值对。

```bash
#赋值
hset key field value
hmset key field value [field2 value2 ...]	#设置key中的多个field/value
#取值
hget key field		#获取key中的指定field的值
hmget key field [field2 ...]	#获取key中的多个field的值
hgetall key		#获取key中的所有key-value
#删除
hdel key field [field2 ...]		#删除一个或多个字段
del key			#删除整个hash
#数值增减
hincrby key field value		#设置key中field的值增加value
#判断指定的key中的field是否存在
hexists key field
#获取key所包含的field的数量
hlen key
#获得所有的key
hkeys key
```

3. 列表（list）

   ​	在Redis中，List类型是按照插入顺序排序的字符串链表。可以在其头部（left）和尾部（right）添加新的元素。在插入时，如果改键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。

```bash
#两端添加
lpush key value1 [value2 value3]
rpush key values [value2 value3]
#查看列表。start、end从0开始计数；也可以为负数，若为-1则表示链表尾部的元素，-2表示倒数第二个，以此类推...
lrange key start end
#两端弹出
lpop key
rpop key
#获取列表中元素的个数
llen key
##扩展命令##
#仅当参数中指定的key存在时，向关联的list头部插入或尾部value。
lpushx key value
rpushx key value
#删除count个值为value的元素，如果count>0，则从头到尾遍历删除；如果count<0，则从尾到头遍历删除；如果count=0，则删除链表中所有等于value的元素。
lrem key count value
#设置链表中的index索引的元素值
lset key index value
#在pivot元素前或者后插入value这个元素
linsert key before | after pivot value
#将链表中的尾部元素弹出并添加到头部
rpoplpush resourcelist destinationlist
```

4. 集合（set）

   ![](C:\Users\华健\Desktop\截图\Set概述.PNG)

```bash
#添加元素
sadd key value [value2 value3 ...] 
#删除元素
srem key member [member2 member3 ...]
#获取set中的所有成员
smembers key
#判断参数中指定的成员是否存在，1表示存在，0表示不存在或者Key本身就不存在
smembers key member
#集合的差集运算
sdiff key1 key2 ... 	#返回属于key1并且不属于key2的元素构成的集合
sdiffstore destination key1 key2 ...
#集合的交集运算
sinter key1 key2 key3 ...	
sinterstore destination key1 key2 key3 ...
#集合的并集运算
sunion key1 key2 key3 ...
sunionstore destination key1 key2 key3 ...
#获取set中成员的数量
scard key
#随机返回set中的一个成员
srandmember key

```

5. 有序集合（sortedset）

![](C:\Users\华健\Desktop\截图\Sortedsort概述.PNG)

```bash
#添加元素
zadd key score member score2 member2 ...
#获得指定成员的分数
zscore key member
#获取集合中的成员数量
zcard key 
#删除元素
zrem key member [member2 ...] 
##范围查询##
#获取集合中索引为start-end的成员，[withscores]参数表明返回的成员包含其分数。
zrange key start end [withscores]
#按照元素分数从大到小的顺序返回索引从start到stop之间的所有元素（包含两端的元素）
zrevrange key start stop [withscores]
#按照排名范围删除元素
zremrangebyrank key start stop
#按照分数范围删除元素
zremrangebyscore key min max
#返回成员在集合中的排名（从小到大）
zrank key member
#返回成员在集合的排名（从大到小）
zrevrank key member
#返回分数在[min,max]之间的成员个数
zcount key min max
#设置指定成员的增加的分数
zincrby key value member
#返回分数在[min,max]的成员并按照分数从低到高排序。[limit offset count]：表示从脚标为offset的元素开始并返回count个成员。
zrangebyscore key min max [withscores] [limit offset count]
```

#### Keys的通用操作

```bash
#获取所有与pattern匹配的key
keys pattern
#删除指定的key
del key1 key2
#判断该key事都存在
exists key
#重命名key
rename key newkey
#设置过期时间，单位：秒
expire key [time]
#获取该key所剩的超时时间。-1：表示没有设置超时；-2：表示已经超时。
ttl key
#获取指定key的类型
type key
```

#### 关于Redis的key的定义，注意几点：

1. key不要太长，最好不要超过1024个字符，这不仅会消耗内存还会降低查找效率；
2. key不要太短，如果太短会降低key的可读性；
3. 在项目中，key最好有一个统一的命名规范。


#### 多数据库

一个Redis实例最多可提供16个数据库，下表从0到15，客户端默认连接第0号数据库，也可以通过select选择连接哪个数据库。

```bash
#连接1号库
select 1
```

将当前库的key移植到1号库中

```bash
move key 1
```

#### 服务器命令

```bash
#测试连接是否存活
ping
#在命令行打印一些内容
echo content
#选择数据库
select index
#退出连接
quit
#返回当前数据库中key的数目
dbsize
#获取服务器的信息和统计
info
#删除当前选择数据库中的所有key
flushdb
#删除所有数据库中的所有key
flushall
```

#### 消息订阅与发布

​	Pub/Sub功能（means Publish, Subscribe）即发布及订阅功能。基于事件的系统中，Pub/Sub是目前广泛使用的通信模型，它采用事件作为基本的通信机制，提供大规模系统所要求的松散耦合的交互模式：订阅者(如客户端)以事件订阅的方式表达出它有兴趣接收的一个事件或一类事件；发布者(如服务器)可将订阅者感兴趣的事件随时通知相关订阅者。熟悉设计模式的朋友应该了解这与23种设计模式中的观察者模式极为相似。 
	同样,Redis的pub/sub是一种消息通信模式，主要的目的是解除消息发布者和消息订阅者之间的耦合,Redis作为一个pub/sub的server,在订阅者和发布者之间起到了消息路由的功能

​	Redis通过publish和subscribe命令实现订阅和发布的功能。订阅者可以通过subscribe向redis server订阅自己感兴趣的消息类型。redis将信息类型称为通道(channel)。当发布者通过publish命令向redis server发送特定类型的信息时，订阅该消息类型的全部订阅者都会收到此消息。

```bash
#订阅一个或多个频道
subscribe channel [channel1 channel2 ...]
#批量订阅频道
psubscribe pattern [pattern2 ...]
#发布消息
publish channel message
#退订一个或多个频道
unsubscribe channel [channel2 ...]
#批量退订频道
punsubscribe pattern [pattern2 ...] 
##查看订阅月发布系统关系##
pubsub subcommamd [argument]
#列出当前的活跃频道
pubsub channels [pattern]
#返回给定频道的订阅者数量，订阅模式的客户端不计算在内
pubsub numsub [chanel-1 ... channel-2]
#返回订阅模式的数量.不是订阅模式的客户端的数量， 而是客户端订阅的所有模式的数量总和。
pubsub numpat
```

#### Pub/Sub在Java中的实践

1. 首先需要一个消息监听器类，继承JedisPubSub类，重写回调方法。

```java
public class RedisMsgPubSubListener extends JedisPubSub {
	//收到消息的回调方法
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("channel:"+channel+"receives message:" + message);
    }
	//订阅频道的回调方法
    @Override
    public void onSubscribe(String channel, int subscribedChannels) {
        System.out.println("channel:"+channel+"is been subscribed:"+ 
        										subscribedChannels);
    }
	//取消订阅的回调方法
    @Override
    public void onUnsubscribe(String channel, int subscribedChannels) {
        System.out.println("channel:"+channel +"is been unsubscribed:"+ 
        											subscribedChannels);
    }
}
```

2. 订阅者

```java
public class TestSubscribe {

    public static void main(String[] args) throws InterruptedException {

        Jedis jedis = new Jedis("192.168.0.8", 6379);
		
        RedisMsgPubSubListener listener =  new RedisMsgPubSubListener();
		//订阅"cctv5"频道，参数加入监听器实例
        jedis.subscribe(listener, "cctv5");

        //other code
        System.out.println("other code");
    }
}
```

注意：**subscribe**是一个阻塞的方法，在取消订阅该频道前，会一直阻塞在这，只有当取消了订阅才会执行下面的other code，参考上面代码，我在onMessage里面收到消息后，调用了this.unsubscribe(); 来取消订阅，这样才会执行后面的other code

3. 发布者

```java
public class TestPublish {

    public static void main(String[] args) throws Exception {

        Jedis jedis = new Jedis("192.168.0.8", 6379);
		//在"cctv5"频道发布消息
        jedis.publish("cctv5", "This is CCTV5");
        Thread.sleep(5000);
        jedis.publish("cctv5", "welcome to watch cctv5");
        Thread.sleep(5000);
        jedis.publish("cctv5", "please focuns on");
    }
}
```

4. 结果

```java
channel:cctv5is been subscribed:1
channel:cctv5receives message :This is CCTV5
channel:cctv5receives message :welcome to watch cctv5
channel:cctv5receives message :please focuns on
```

#### Redis事务

- multi：开启事务用于标记事务的开始，其后执行的命令将被存入命令队列，知道执行EXEC时，这些命令才会被原子性的执行，类似与关系数据库中的：begin transaction
- exec：提交事务，类似与关系数据库中的：commit
- discard：事务回滚，类似与关系数据库中的：rollback

事务特征：

1. 在事务中的所有命令都会被串行化的顺序执行，事务执行期间，Redis不会再为其它客户端的请求提供任何服务，从而保证了事务中的所有命令被原子的执行。
2. Redis事务中如果有一条命令执行失败，其后的命令任然会被继续执行。
3. 在事务开启之前，如果客户端与服务器之间出现通讯故障并导致网络断开，其后所有待执行的语句都将不会被服务器执行。然而如果网络中中断事件是发生在客户端执行EXEC命令之后，那么该事务中的所有命令都会被服务器执行。
4. 当使用Append-Only模式时，Redis会通过调用系统函数write将该事务内的所有写操作在本次调用中全部写入磁盘。然而如果在写入的过程中出现系统崩溃，如电源故障导致的宕机，那么此时也许只有部分数据被写入磁盘，而另外一部分数据却已经丢失。Redis服务器在重新启动时执行一系列必要的一致性检测，一旦发现类似问题，就会立即退出并给出相应的错误提示。此时，我们就要充分利用Redis工具包中提供的redis-check-aof工具，该工具可以帮助我们定位到数据不一致的错误，并将已经写入的部分数据进行回滚。修复之后我们就可以再次重新启动Redis服务器了。

#### Redis持久化的方式

1. RDB持久化（默认支持，无需配置）

   该机制是指定在指定的时间间隔内将内存中的数据集快照写入磁盘。

   快照参数在redis.conf中配置

   ```bash
   save 9000 1		#每900秒(15分钟)至少有1个key发生变化，则dump内存快照。
   save 300 10		#每300秒(5分钟)至少有10个key发生变化，则dump内存快照。
   save 60 10000	#每60秒(1分钟)至少有10000个key发生变化，则dump内存快照。

   #保存文件名称
   dbfilename dump.rdb
   #保存位置
   dir ./
   ```

2. AOF持久化

   该机制将以日志的形式记录服务器所处理的每一个写操作，在Redis服务器启动之后会读取该文件来重新构建数据库，以保证启动后数据库中的数据是完整的。

   配置AOF，在redis.conf中修改参数

   ```bash
   appendonly yes	#开启AOF，默认关闭
   appendfilename "appendonly.aof"		#日志文件名
   # appendfsync always	#每次有数据修改发生时都会写入AOF文件
   appendfsync everysec	#每秒钟同步一次，该策略为AOF的缺省策略
   # appendfsync no		#从不同步。高效但是数据不会被持久化
   ```

注：若不满足重写条件时，可以手动重写，命令：bgrewriteaof。

数据库还原示例：

- 步骤1：开启AOF，重启Redis。
- 步骤2：在窗口1进行若干操作。
- 步骤3：清空数据库，flushall。
- 步骤4：在窗口2，关闭Redis，修改appendonly.aof文件，将flushall命令删除。
- 步骤5：在窗口1启动Redis，然后查询数据库内容，发现数据库已还原。

3. 无持久化

我们可以通过配置的方式禁用Redis服务器的持久化功能，这样我们就可以将Redis视为一个功能加强版的memcached了。

4. redis可以同时使用RDB和AOF。









#### Redis常用数据类型

![](C:\Users\华健\Pictures\Redis常用数据类型.PNG)

Redis+Cookie+Jackson+Filter实现单点登录

封装RedisPool类

```java
public class RedisPool {
    private static JedisPool pool;
    private static Integer maxTotal = 
        Integer.parseInt(PropertiesUtil.getProperty("redis.max.total", ""));
    private static Integer maxIdle = 
        Integer.parseInt(PropertiesUtil.getProperty("redis.max.idle", ""));
    private static Integer minIdle = 
        Integer.parseInt(PropertiesUtil.getProperty("redis.min.idle", ""));

    private static Boolean testOnBorrow = 
        Boolean.parseBoolean(PropertiesUtil.getProperty("redis.test.borrow", ""));
    private static Boolean testOnReturn = 
        Boolean.parseBoolean(PropertiesUtil.getProperty("redis.test.return", ""));

    private static String redisIp = PropertiesUtil.getProperty("redis.ip");
    private static Integer redisPort = Integer.parseInt(PropertiesUtil.getProperty("redis.port"));

    private static void initPool() {
        JedisPoolConfig config = new JedisPoolConfig();

        config.setMaxTotal(maxTotal);
        config.setMaxIdle(maxIdle);
        config.setMinIdle(minIdle);

        config.setTestOnBorrow(testOnBorrow);
        config.setTestOnReturn(testOnReturn);

        config.setBlockWhenExhausted(true);

        pool = new JedisPool(config, redisIp, redisPort, 1000*2);
    }
	//初始化类的时候初始化连接池
    static{
        initPool();
    }

    public static Jedis getJedis(){
        return pool.getResource();
    }
    
    public static void returnBrokenResource(Jedis jedis) {
        pool.returnBrokenResource(jedis);
    }

    public static void returnResource(Jedis jedis) {
        pool.returnResource(jedis);
    }
}
```

封装Redis API

```java
public class RedisPoolUtil {

    private  static Logger logger = LoggerFactory.getLogger(RedisPoolUtil.class);

    //重新设置key的有效期，单位：秒
    public static Long expire(String key, int exTime) {
        Jedis jedis = null;
        Long result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.expire(key, exTime);
        } catch (Exception e) {
            logger.error("expire key:{} value:{} error", key, e);
            RedisPool.returnBrokenResource(jedis);
            return result;
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    public static String setEx(String key, String value, int exTime) {
        Jedis jedis = null;
        String result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.setex(key, exTime, value);
        } catch (Exception e) {
            logger.error("setex key:{} value:{} error", key, value, e);
            RedisPool.returnBrokenResource(jedis);
            return result;
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    public static String set(String key, String value) {
        Jedis jedis = null;
        String result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.set(key, value);
        } catch (Exception e) {
            logger.error("set key:{} value:{} error", key, value, e);
            RedisPool.returnBrokenResource(jedis);
            return result;
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    public static String get(String key) {
        Jedis jedis = null;
        String result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.get(key);
        } catch (Exception e) {
            logger.error("get key:{} error", key, e);
            RedisPool.returnBrokenResource(jedis);
            return result;
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    public static Long del(String key) {
        Jedis jedis = null;
        Long result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.del(key);
        } catch (Exception e) {
            logger.error("del key:{} error", key, e);
            RedisPool.returnBrokenResource(jedis);
            return result;
        }
        RedisPool.returnResource(jedis);
        return result;
    }
}
```

序列化与反序列化工具类：JsonUtil

```java
public class JsonUtil {

    private static Logger logger = LoggerFactory.getLogger(JsonUtil.class);

    private static ObjectMapper objectMapper = new ObjectMapper();

    static{
        //对象的所有字段全部列入
        objectMapper.setSerializationInclusion(JsonSerialize.Inclusion.ALWAYS);
        //取消默认转换timestamps形式
        objectMapper.configure(SerializationConfig.Feature.WRITE_DATES_AS_TIMESTAMPS, false);
        //忽略空Bean转json的错误
        objectMapper.configure(SerializationConfig.Feature.FAIL_ON_EMPTY_BEANS, false);
        //所有的日期格式都统一为一下的样式，即yyyy-MM-dd HH:mm:ss
        objectMapper.setDateFormat(new SimpleDateFormat(DateTimeUtil.STANDARD_FORMAT));
        //忽略在json字符差U呢中存在，但是在java对象中不存在对应属性的情况，防止错误
        objectMapper.configure(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }
	//序列化
    public static <T> String obj2String(T obj) {
        if (obj == null) {
            return null;
        }
        try {
            return obj instanceof String ? (String)obj : objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            logger.warn("Parse object to String error", e);
            return null;
        }
    }
	//序列化方法2
    public static <T> String obj2StringPretty(T obj) {
        if (obj == null) {
            return null;
        }
        try {
            return obj instanceof String ? (String)obj : 
            			objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(obj);
        } catch (Exception e) {
            logger.warn("Parse object to String error", e);
            return null;
        }
    }

    //反序列化方法1
    public static <T> T string2Obj(String str, Class<T> clazz) {
        if(StringUtils.isEmpty(str) || clazz  == null) {
            return null;
        }
        try {
            return clazz.equals(String.class) ? (T)str : objectMapper.readValue(str, clazz);
        } catch (IOException e) {
            logger.warn("Parse String to Object error", e);
            return null;
        }
    }
    //反序列化方法2
    public static <T> T string2Obj(String str, TypeReference<T> typeReference) {
        if(StringUtils.isEmpty(str) || typeReference  == null) {
            return null;
        }
        try {
            return (T) (typeReference.getType().equals(String.class) ? str : 
                        				objectMapper.readValue(str, typeReference));
        } catch (IOException e) {
            logger.warn("Parse String to Object error", e);
            return null;
        }
    }
    //反序列化方法3
    public static <T> T string2Obj(String str, Class<?> collectionClass, Class<?>... elements) {
        JavaType javaType = 
            objectMapper.getTypeFactory().constructParametricType(collectionClass, elements);
        try {
            return objectMapper.readValue(str, javaType);
        } catch (IOException e) {
            logger.warn("Parse String to Object error", e);
            return null;
        }
    }   
}
```

封装的CookieUtil

```java
public class CookieUtil {

    private static Logger logger = LoggerFactory.getLogger(CookieUtil.class);

    private final static String COOKIE_DOMAIN = ".happymall.com";
    private final static String COOKIE_NAME = "mmall_login_token";

    public static void writeLoginToken(HttpServletResponse response, String token) {
        Cookie ck = new Cookie(COOKIE_NAME, token);
        ck.setDomain(COOKIE_DOMAIN);
        ck.setPath("/");
        ck.setHttpOnly(true);
        ck.setMaxAge(-1);   //-1代表永久
        logger.info("write cookieName:{},cookieValue:{}", ck.getName(), ck.getValue());
        response.addCookie(ck);
    }

    public static String readLoginToken(HttpServletRequest request) {
        Cookie[] cks = request.getCookies();
        if(cks != null) {
            for(Cookie ck : cks) {
                if(StringUtils.equals(ck.getName(), COOKIE_NAME)) {
                    logger.info("return cookieName:{},cookieValue:{}", 
                                ck.getName(), ck.getValue());
                    return ck.getValue();
                }
            }
        }
        return null;
    }

    public static void delLoginToken(HttpServletRequest request, 
                                     HttpServletResponse response) {
        Cookie[] cks = request.getCookies();
        if(cks != null) {
            for(Cookie ck : cks) {
                if(StringUtils.equals(ck.getName(), COOKIE_NAME)) {
                    logger.info("return cookieName:{},cookieValue:{}", 
                                ck.getName(), ck.getValue());
                    ck.setDomain(COOKIE_DOMAIN);
                    ck.setPath("/");
                    ck.setMaxAge(0);   //0代表删除
                    logger.info("del cookieName:{},cookieValue:{}", 
                                ck.getName(), ck.getValue());
                    response.addCookie(ck);
                    return;
                }
            }
        }
    }
}
```

重置Redis中用户信息时长，用过滤器

```java
public class SessionExpireFilter implements Filter{

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                         FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String loginToken = CookieUtil.readLoginToken(httpServletRequest);
        if(StringUtils.isNotEmpty(loginToken)) {
            String userJsonStr = RedisPoolUtil.get(loginToken);
            User user = JsonUtil.string2Obj(userJsonStr, User.class);
            if(user != null) {
                //每次请求都重置Redis中用户信息的时长
                RedisPoolUtil.exp
                    ire(loginToken, Const.RedisCacheExtime.REDIS_SESSION_EXTIME);
            }
        }
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
    }
}
```

在xml中注册过滤器







封装分布式ShardedJedisPool

```java
public class RedisSharedPool {
    private static ShardedJedisPool pool;
    private static Integer maxTotal = 
        Integer.parseInt(PropertiesUtil.getProperty("redis.max.total", ""));
    private static Integer maxIdle = 
        Integer.parseInt(PropertiesUtil.getProperty("redis.max.idle", ""));
    private static Integer minIdle = 
        Integer.parseInt(PropertiesUtil.getProperty("redis.min.idle", ""));

    private static Boolean testOnBorrow = 
        Boolean.parseBoolean(PropertiesUtil.getProperty("redis.test.borrow", ""));
    private static Boolean testOnReturn = 
        Boolean.parseBoolean(PropertiesUtil.getProperty("redis.test.return", ""));

    private static String redis1Ip = PropertiesUtil.getProperty("redis1.ip");
    private static Integer redis1Port = 
        Integer.parseInt(PropertiesUtil.getProperty("redis1.port"));
    private static String redis2Ip = PropertiesUtil.getProperty("redis2.ip");
    private static Integer redis2Port = 
        Integer.parseInt(PropertiesUtil.getProperty("redis2.port"));

    private static void initPool() {
        JedisPoolConfig config = new JedisPoolConfig();

        config.setMaxTotal(maxTotal);
        config.setMaxIdle(maxIdle);
        config.setMinIdle(minIdle);

        config.setTestOnBorrow(testOnBorrow);
        config.setTestOnReturn(testOnReturn);

        config.setBlockWhenExhausted(true);

        JedisShardInfo info1 = new JedisShardInfo(redis1Ip, redis1Port, 1000*2);
        JedisShardInfo info2 = new JedisShardInfo(redis2Ip, redis2Port, 1000*2);

        List<JedisShardInfo> jedisShardInfoList = new ArrayList<JedisShardInfo>(2);
        jedisShardInfoList.add(info1);
        jedisShardInfoList.add(info2);

        pool = new ShardedJedisPool(config, jedisShardInfoList, Hashing.MURMUR_HASH, 
                                    Sharded.DEFAULT_KEY_TAG_PATTERN);
    }

    static{
        initPool();
    }

    public static ShardedJedis getJedis(){
        return pool.getResource();
    }

    public static void returnBrokenResource(ShardedJedis jedis) {
        pool.returnBrokenResource(jedis);
    }

    public static void returnResource(ShardedJedis jedis) {
        pool.returnResource(jedis);
    }
}
```

