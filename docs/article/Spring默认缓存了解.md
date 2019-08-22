# Spring默认缓存了解

> @EnableCaching、@Cacheable、@CachePut、@CacheEvict

## 使用步骤

1. 引入spring-boot-starter-cache模块
2. 使用`@EnableCaching`开启缓存
3. 使用缓存对应的注解
   - @Cacheable
   - @CacheEvict
   - @CachePut

## @Cacheable运行流程

**@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值做为key来查询缓存，
如果没有就运行方法来将结果放入缓存，以后再来调用就可以直接使用缓存中的数据。**

## @Cacheable注解参数

- cacheNames/value  
  用于指定缓存组件的名称，将方法的结果返回到相应的缓存中。
- key/keyGenerator  
  - key  
    缓存的key，用来查询缓存值，默认为方法参数的id值。也可以通过spEL表达式表示。  
    key = "#root.methodName+'['+#id+']'" 可以指定key值为getEmp[1]
  - keyGenerator  
    自定义keyGenerator组件，添加到容器中的。并且在@Bean注解中指定名称，用于引用。
- cacheManager/cacheResolver  
  缓存管理器/缓存解析器
- condition/unless  
  - condition：符合条件进行缓存。
  - unless：否定缓存，满足条件不进行缓存
- sync  
  使用异步模式

## @CachePut

@CachePut注解，标注数据库更新方法；即调用方法对数据库进行更改，同时也更新缓存。  
注明`cacheNames/value `参数，指明缓存组件；注明`key`参数，用于指定更新缓存值。

## @CacheEvict

@CacheEvict，标注删除方法，进行数据库数据删除的同时，删除缓存中名称为`cacheNames/value`组件中`key`对应的缓存数据。