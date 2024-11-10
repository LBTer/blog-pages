---
title:       "redis面试问题"
description: "收集面试问题集并总结"
date:        2024-08-29T00:00:00
author:     "LBT"
tags:        ["redis", "interview"]
categories:  ["Tech" ]
---
## 1、redis面试资料整理
[redis20道面试题](https://github.com/CoderLeixiaoshuai/java-eight-part/blob/master/docs/redis/%E7%9C%8B%E5%AE%8C%E8%BF%9920%E9%81%93Redis%E9%9D%A2%E8%AF%95%E9%A2%98%EF%BC%8C%E9%98%BF%E9%87%8C%E9%9D%A2%E8%AF%95%E5%8F%AF%E4%BB%A5%E7%BA%A6%E8%B5%B7%E6%9D%A5%E4%BA%86.md) \
**推荐**[小林redis常见面试题](https://www.xiaolincoding.com/redis/base/redis_interview.html)，还列出了7.0的变化 \
[redis常见面试题总结](https://javaguide.cn/database/redis/redis-questions-01.html) \
[redis56题](https://javabetter.cn/sidebar/sanfene/redis.html) \
[redis面试题+操作](https://github.com/zxlrise/StudyNotes/blob/main/%E6%95%B0%E6%8D%AE%E5%BA%93/Redis/Redis%E9%9D%A2%E8%AF%95/Redis%E9%9D%A2%E8%AF%95%E9%A2%98.md)

## 2、典型书面问题
- 1、redis是什么？可以做什么？
- 2、为什么redis块
    出自[Alex xu](https://x.com/alexxubyte/status/1498703822528544770)
![]()

## 3、典型redis应用设计

## 4、redis问题
### redis setnx + expire 有什么缺点，如何优化。
在 Redis 中，`SETNX` 命令用于设置键值对，仅当键不存在时。`EXPIRE` 命令用于为键设置生存时间。将这两个命令组合使用的场景通常是尝试设置一个键，并为其设置一个过期时间，如果键已经存在，则不执行任何操作。

#### 缺点：

1. **非原子操作**：
   - 使用 `SETNX` 和 `EXPIRE` 是两个分开的命令，这意味着在高并发的情况下，可能会有两个客户端几乎同时检查到键不存在，然后都尝试设置它，导致键被覆盖。

2. **竞态条件**：
   - 在执行 `SETNX` 和 `EXPIRE` 之间，如果其他客户端修改了键，会导致不一致的行为。

3. **延迟问题**：
   - 如果两个命令之间有延迟，可能会影响到数据的一致性。

4. **效率问题**：
   - 执行两个命令而不是一个原子操作会增加网络开销和处理时间。

#### 优化方法：

1. **使用 `SETEX` 命令**：
   - Redis 2.6.12 版本引入了 `SETEX` 命令，它将设置键值对和设置过期时间的操作合并为一个原子操作。

   ```sh
   SETEX key seconds value
   ```

2. **使用 `PSETEX` 命令**：
   - 类似于 `SETEX`，`PSETEX` 命令以毫秒为单位设置键的过期时间。

3. **使用事务（MULTI/EXEC）**：
   - 将 `SETNX` 和 `EXPIRE` 命令包装在事务中，确保它们作为单个原子操作执行。

   ```sh
   MULTI
   SETNX key value
   EXPIRE key seconds
   EXEC
   ```

4. **使用 `SET` 命令的现代替代品**：
   - Redis 4.0 引入了新的 `SET` 命令，它支持一系列选项，包括条件设置和过期时间。

   ```sh
   SET key value NX PX milliseconds
   ```

   - `NX` 选项表示仅当键不存在时设置键。
   - `PX` 选项用于以毫秒为单位设置过期时间。

5. **使用 Lua 脚本**：
   - 使用 Lua 脚本来保证 `SETNX` 和 `EXPIRE` 的原子性。

   ```sh
   EVAL "if redis.call('setnx', KEYS[1], ARGV[1]) == 1 then return redis.call('expire', KEYS[1], ARGV[2]) else return 0 end" 1 key value seconds
   ```

6. **监控和测试**：
   - 对关键操作进行适当的监控和测试，确保在高并发环境下应用的行为符合预期。

通过使用这些优化方法，你可以确保操作的原子性，减少竞态条件的风险，并提高应用程序的效率和可靠性。
