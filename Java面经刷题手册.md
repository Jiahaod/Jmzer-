## Java基础相关

----

### Java的拆装箱

基本数据类型遍历不需要通过 new 关键字进行创建，不会在堆上进行创建，而是在栈上进行创建，而是直接在栈中存储。

包装类

| 基本数据类型 |  包装类   |
| :----------: | :-------: |
|     byte     |   Byte    |
|   boolean    |  Boolean  |
|    short     |   Short   |
|     char     | Character |
|     int      |  Integer  |
|     long     |   Long    |
|    float     |   Float   |
|    double    |  Double   |

为什么我们需要包装类呢？

1、集合中我们无法放置基本数据类型，集合中的元素我们要求的是一个Object。包装类就是将基本数据类型进行包装，使其具有对象的性质。

注：包装类的比较最好使用到equals()

### 接口和抽象类的区别

抽象类能够被继承，接口被实现，抽象类可以说是特殊的类，不能被实例化。抽象类中可以有未实现的方法申明

### 源码相关

#### List

##### ArrayList

1. 扩容机制，容量大小默认的是10，当我们确定了容量大小的适合，我们将最小容量和默认容量进行比较，小于10将10设位容量，当大于10我们需要进行进一步的扩容。扩容是原来大小的1.5倍。源码如下，当扩容后还是比我们要求的小，我们扩容到我们所要求的大小

```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

2. 如果要使用多线程，考虑线程安全，使用Collections.synchronizedList

```
List list = Collections.synchronizedList(new ArrayList(...))
```

3. contains(Object o) 

   使用的是return indexOf(o), 改object在ArrayList是否有索引。

4. ArrayList中 set、remove(的一个重载)   return的是一个oldValue

   ```
   public E set(int index, E element) {
       rangeCheck(index);
   
       E oldValue = elementData(index);
       elementData[index] = element;
       return oldValue;
   }
   ```

阅读到714行

private boolean batchRemove(Collection<?> c, boolean complement)

批量删除。

是用来将指定ArrayList中的包含Collection<?> c中的元素进行remove掉(或者反向操作)，true时，则实行remove掉。

####  StringBuffer 和 StringBuilder

StringBuffer线程安全，StringBuffer的public方法都是sychronized修饰的。

StringBuffer每次获取toString都会直接使用缓存区的toStringCache来构造字符串，而StringBuilder每次都是复制一个字符数组，再构造一个字符串。

StringBuilder性能更好。

## JVM

### 垃圾回收算法

1. 标记-清除算法
2. 复制算法
3. 标记-压缩算法

  ## 产品上线后的问题

OOM问题的定位(Out of memery 内存溢出)

Java服务OOM，最常见的原因为：

（1）有可能是内存分配确实过小，而正常业务需要使用更大的内存；

（2）某一个对象被频繁申请，却没有释放，内存不断泄露，导致内存耗尽；

（3）某一个资源被不断申请，系统资源耗尽，例如：不断创建线程，不断发起网络连接

JVM中使用jmap -heap 10765  确定内存是否分配小了。

jmp -histo:live 10765 | mire   查看最耗内存的对象   

可能：

- 申请完资源后，未调用close()或dispose()释放资源
- 消费者消费速度慢(或停止消费了)，而生产者不断往队列中投递任务，导致队列中任务累积过多

## 锁

-----

### 本地锁和分布式锁

本地锁有sychronized和lock

分布式情况下（多JVM），线程A和线程B很可能不是在同一JVM中，这样线程锁就无法起到作用了，这时候就要用到分布式锁来解决。

分布式锁

1. 基于数据库实现分布式锁

    在数据库中创建一个表，并在**字段上创建唯一索引**，想要执行某个方法，就使用这个方法名向表中插入数据，成功插入则获取锁，执行完成后删除对应的行数据释放锁。

2. 基于缓存（Redis等）实现分布式锁

3. 基于Zookeeper实现分布式锁

   

### 数据库常用锁

表级锁定，行级锁定和页级锁定。

## Spring

### IOC

控制翻转

### AOP

## Mybatis

一级缓存

缓存失效的情况：
1、查询不同的东西
2、增删改操作，可能会改变原来的数据，会刷新缓存

```
sqlSession.clearCache();    //手动清理缓存
```
一级缓存是默认开启的，只在一次SqlSession中有效，就是拿到连接到关闭连接这个区间段

二级缓存

- 二级缓存也叫全局缓存，一级缓存作用域太低，所以诞生了二级缓存
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存。
- 工作机制

1、一个会话查询一条数据，这个数据就会被防止当前会话的一级缓存中；

2、如果会话关闭了，这个会话对应的一级缓存就不存在，我们需要的是
会话关闭了，一级缓存中的数据会被保存到二级缓存。

3、新会话查询信息，可以从二级缓存中获取内容。

步骤：
1、开启全局缓存
```
<setting name="cacheEnabled" value="true"/>
```
2、在要使用二级缓存的Mapper中开启,并自定义参数
```
<cache eviction="FIFO"
           flushInterval="60000"
           size="512"
           readOnly="true"/>
```
只要开启了二级缓存，在同一个Mapper中就有效

先看二级缓存有没有，一级缓存有没有，再看数据库

## Redis

### 五种数据类型

string     list   hash  set    zset

### 三种特殊的数据类型

bitmaps  hyperloglogs  geospatial

### 持久化机制

RDB AOF

RDB持久化是指用数据集快照的方式记录redis数据库的所有键值对。

 两个命令：SAVE命令会阻塞主进程来完成写文件，BGSAVE命令会创建子进程来完成写文件，主进程会继续处理命令。

 优点：

　　1.只有一个文件dump.rdb，方便持久化。

　　2.容灾性好，一个文件可以保存到安全的磁盘。

　　3.性能最大化，fork子进程来完成写操作，让主进程继续处理命令，所以是IO最大化。

　　4.相对于数据集大时，比AOF的启动效率更高。

 缺点：

　　1.数据安全性低，通过配置save参数来达到定时的写快照，比如 每900 秒有1个键被修改就进行一次快照，每600秒至少有10个键被修改进行快照，每30秒有至少10000个键被修改进行记录。所以如果当服务器还在等待写快照时出现了宕机，那么将会丢失数据。

　　2.fork子进程时可能导致服务器停机1秒，数据集太大。



AOF持久化是指所有的命令行记录以redis命令请求协议的格式保存为aof文件。

 优点：

　　1.数据安全，aof持久化可以配置appendfsync属性，有always，每进行一次命令操作就记录到aof文件中一次；everySec，就是每秒内进行一次文件的写操作；no就是不进行aof文件的写操作。

　　2.通过append模式写文件，即使中途服务器宕机，可以通过redis-check-aof工具解决数据一致性问题。

　　3.AOF机制的rewrite模式，用来将过大的aof文件缩小，实现原理是将所有的set 通过一句set 命令总结，所有的SADD命令用总结为一句，这样每种命令都概括为一句来执行，就可以减少aof文件的大小了。（注意，在重写的过程中，是创建子进程来完成重写操作，主进程每个命令都会在AOF缓冲区和AOF重写缓冲区进行保存，这样旧版aof文件可以实现数据最新，当更新完后将重写缓冲区中的数据写入新的aof文件中然后就可以将新的文件替换掉旧版的文件。

