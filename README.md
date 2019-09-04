# springboot-redis-cache
在springboot中使用cache

# info
从3.1开始，Spring引入了对Cache的支持。其使用方法和原理都类似于Spring对事务管理的支持。Spring Cache是作用在方法上的，其核心思想是这样的：当我们在调用一个缓存方法时会把该方法参数和返回结果作为一个键值对存放在缓存中，等到下次利用同样的参数来调用该方法时将不再执行该方法，而是直接从缓存中获取结果进行返回。所以在使用Spring Cache的时候我们要保证我们缓存的方法对于相同的方法参数要有相同的返回结果。

       使用Spring Cache需要我们做两方面的事：
           1.声明某些方法使用缓存
           2. 配置Spring对Cache的支持
       和Spring对事务管理的支持一样，Spring对Cache的支持也有基于注解和基于XML配置两种方式。下面我们先来看看基于注解的方式。
       
 # 1 基于注解的支持
    Spring为我们提供了几个注解来支持Spring Cache。其核心主要是@Cacheable和@CacheEvict。使用@Cacheable标记的方法在执行后Spring Cache将缓存其返回结果，而使用@CacheEvict标记的方法会在方法执行前或者执行后移除Spring Cache中的某些元素。下面我们将来详细介绍一下Spring基于注解对Cache的支持所提供的几个注解。

## 1.1 @Cacheable
       @Cacheable可以标记在一个方法上，也可以标记在一个类上。当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。对于一个支持缓存的方法，Spring会在其被调用后将其返回值缓存起来，以保证下次利用同样的参数来执行该方法时可以直接从缓存中获取结果，而不需要再次执行该方法。Spring在缓存方法的返回值时是以键值对进行缓存的，值就是方法的返回结果，至于键的话，Spring又支持两种策略，默认策略和自定义策略，这个稍后会进行说明。需要注意的是当一个支持缓存的方法在对象内部被调用时是不会触发缓存功能的。@Cacheable可以指定三个属性，value、key和condition。

|参数|解释|example|
|-----------|--------------------------------|------------------------------------|
|value|缓存的名称，在 spring 配置文件中定义，必须指定至少一个| 例如: @Cacheable(value=”mycache”) @Cacheable(value={”cache1”,”cache2”}|

|key|缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合|@Cacheable(value=”testcache”,key=”#userName”)|

|condition|缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者  false，只有为 true 才进行缓存|  @Cacheable(value=”testcache”,condition=”#userName.length()>2”)|
