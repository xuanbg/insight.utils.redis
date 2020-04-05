# 使用说明

## 目录

- [概述](#概述)
- [功能模块](#功能模块)
  - [Generator](#Generator)
  - [LockHandler](#LockHandler)
  - [Redis](#Redis)

## 概述

此工具包为 **insight.utils** 成员，主要提供 **Redis** 操作相关功能。

## 功能模块

### Generator

自定义格式编码生成相关类，提供两种编码生成方法。可用于业务单号，例如订单号的生成。生成编码时需先获取基于 Redis 的分布式锁，以保证编码的唯一性。且生成的流水号以编码分组标识为 Key 存储于 Redis 中。

参数如下：

|参数类型|参数名|参数说明|
| ----- | ----- | ----- |
|String|format|编码格式。编码格式支持在自定义字符串中嵌入日期和流水号，日期格式为年(yyyy/yy)、月(MM)、日(dd)的任意组合。流水号格式为#L，L为位流水号位数(2-8)。|
|String|group|唯一编码分组标识。每个分组都有独立的流水号，一般使用业务名称作为标识即可。|
|Boolean|isEncrypt|是否加密流水号。流水号加密后变得无序，无法通过流水号得知业务量。|

> 注：流水号到达位数能支持的最大值后重新从0开始，使用者需要相对嵌入的日期格式设定一个足够的流水号空间，以免编码重复。如日期格式为yyyyMMdd，只需保证一天内流水号不会用尽即可。如日期格式为yyyyMM，则需保证一个月内流水号不会用尽才可以。

代码示例：

```Java
String formart = "O-yyyyMMdd-#5";
String group = "Order";

// code为一个流水加密的编码: O-20200405-42386
String code = Generator.newCode(format, group);
```

```Java
String formart = "O-yyyyMMdd-#5";
String group = "Order";

// code为一个流水不加密的编码: O-20200405-00001
String code = Generator.newCode(format, group, false);
```

### LockHandler

分布式锁相关类，提供基于 Redis 的分布式锁。示例代码见 Generator 类。

### Redis

Redis操作帮助类，封装了基于 StringRedisTemplate 的 Redis 数据基本操作方法。

代码示例：

```Java
// 对string key 进行操作的相关方法
String key = "redis key";

// Redis中是否存在key
boolean keyIsExists = Redis.hasKey(key);

TimeUnit timeUnit = TimeUnit.SECONDS;

// 获取Key过期时间
long keyExpire = Redis.getExpire(key, timeUnit);

// 设置Key过期时间
Redis.setExpire(key, keyExpire, timeUnit);

// 从Redis删除指定的key
Redis.deleteKey(key);

// 从Redis读取指定key的值
String value = Redis.get(key);

// 以键值对方式保存数据到Redis
Redis.set(key, value);

// 以键值对方式保存数据到Redis，并设置过期时间
Redis.set(key, value, keyExpire, timeUnit);


// 对hash key 进行操作的相关方法
String field = "field";

// Redis中是否存在hash key
boolean keyIsExists = Redis.hasKey(key, field);

// 从Redis读取指定key和field的值
String value = Redis.get(key, field);

// 以Hash方式保存数据到Redis
Redis.set(key, field, value);

// 从Redis删除指定的key
Redis.deleteKey(key, field);


// 对set 进行操作的相关方法
String member = "member";

// 是否指定key的成员
boolean isMember = Redis.isMember(key, member);

// 从Redis读取指定key的全部成员
List<String> members = Redis.getMembers(key);

// 以Set方式保存数据到Redis
Redis.add(key, member);

// 删除指定key的Set成员
long count = Redis.remove(key, member);
```
