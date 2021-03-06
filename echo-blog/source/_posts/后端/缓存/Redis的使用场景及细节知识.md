---
title: 缓存
categories:
- 后端/缓存
tags: redis
date:
---

> 使用场景：针对短期内不经常发生改变的数据，需要频繁查询的数据放入到redis缓存中；
## redis的数据类型：
    - string
    - list（列表）
    - hash
    - set（集合）
    - zset(sorted set有序集合)；
## 解决redis缓存同步问题：
   思路：在数据库更新之前将redis缓存库清除，查询时将更新后的数据库数据查询并同步到redis，为了防止更新后的redis出现宕机发生数据丢失的情况，我们需要搭建redis集群来保证数据的完整性；同时我们在更新redis时，要做好异步定时任务，定期检测redis工作是否异常，保证redis正常启动后能够更新到最新数据；
   拓展：在上面处理redis缓存同步的情况中，可能会出现高并发的问题，这个时候如果不加以处理的话，是很难保证更新到的数据是最新的，最好是使用消息队列技术对请求进行对列化。然后redis再一一对这些数据进行处理；
## redis持久化技术：AOF、RDB
   - AOF:半持久化技术(定时持久化）：可能会发生数据丢失；
   - RDB：时时持久化技术，性能相对较差，读写频繁，同结构数据的文件大小比aof型更大，但是数据完整性较高；