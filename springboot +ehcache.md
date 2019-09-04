# springboot +ehcache

## Springboot整合EhCache

整合步骤：

1. 添加pom文件maven依赖
2. 配置ehcache.xml
3. 开启缓存支持
4. 项目中使用

```java
<!--开启 cache 缓存 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- ehcache缓存 -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.9.1</version><!--$NO-MVN-MAN-VER$ -->
</dependency>

```

## 配置ehcache.xml

默认自动加载resources目录下的ehcache.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <!--timeToIdleSeconds 当缓存闲置n秒后销毁 -->
    <!--timeToLiveSeconds 当缓存存活n秒后销毁 -->
    <!-- 缓存配置 
        name:缓存名称。 
        maxElementsInMemory：缓存最大个数。 
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。 
        timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。 
        timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。 
        overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。 diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。 
        maxElementsOnDisk：硬盘最大缓存个数。 
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk 
        store persists between restarts of the Virtual Machine. The default value 
        is false. 
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。  memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是 
LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。 
        clearOnFlush：内存数量最大时是否清除。 -->
    <!-- 磁盘缓存位置 -->
    <diskStore path="java.io.tmpdir" />
    <!-- 默认缓存 -->
    <defaultCache 
        maxElementsInMemory="10000" 
        eternal="false"
        timeToIdleSeconds="120" 
        timeToLiveSeconds="120" 
        maxElementsOnDisk="10000000"
        diskExpiryThreadIntervalSeconds="120" 
        memoryStoreEvictionPolicy="LRU">

        <persistence strategy="localTempSwap" />
    </defaultCache>

    <!-- 测试 -->
    <cache name="GoodsType" 
        eternal="false" 
        timeToIdleSeconds="2400"
        timeToLiveSeconds="2400" 
        maxEntriesLocalHeap="10000"
        maxEntriesLocalDisk="10000000" 
        diskExpiryThreadIntervalSeconds="120"
        overflowToDisk="false" 
        memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>

```



对于EhCache的配置文件也可以通过application.yml文件中使用spring.cache.ehcache.config属性来指定，比如：

```yml
spring:
  cache:
    ehcache:
      config: classpath:ehcache.xml
```



## ehcache.xml配置文件详解

* diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。
* defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
* name:缓存名称。
* maxElementsInMemory:缓存最大数目
* maxElementsOnDisk：硬盘最大缓存个数。
* eternal:对象是否永久有效，一但设置了，timeout将不起作用。
* overflowToDisk:是否保存到磁盘，当系统宕机时
* timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
* timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
* diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
* diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
  diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
* memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
* clearOnFlush：内存数量最大时是否清除。

```
FIFO，first in first out，先进先出。 
LFU， Less Frequently Used，一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。 
LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。

```

## 使用示例

```java
// 参考 springboot  redis cache

@Repository
@CacheConfig(cacheNames = "GoodsType")
public class GoodsTypeDaoImpl {

    @Cacheable
    public String save(String typeId) {
        System.out.println("save()执行了=============");
        return "模拟数据库保存";
    }

    @CachePut
    public String update(String typeId) {
        System.out.println("update()执行了=============");
        return "模拟数据库更新";
    }

    @CacheEvict
    public String delete(String typeId) {
        System.out.println("delete()执行了=============");
        return "模拟数据库删除";
    }

    @Cacheable
    public String select(String typeId) {
        System.out.println("select()执行了=============");
        return "模拟数据库查询";
    }
}
```

