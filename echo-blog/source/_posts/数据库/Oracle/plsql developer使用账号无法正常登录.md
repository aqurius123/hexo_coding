---
title: plsql developer
categories:
- 数据库
tags: plsql developer
date: 2021-02-14
---

> 在自己学习Oracle相关数据库知识时，总是喜欢进行实操来进行，这样就不得不借助数据库连接工具来实现——plsql developer；在使用的过程中，不像Mysql数据库一样可以直接使用，直接选择数据库就可以使用。必须需要我们登录账号才能进行数据库的相关操作；但是有时可能由于系统更新或者plsql developer过了试用期，导致之前可以进行登录的账号无法再次进行登录从而无法进行任何操作；

## 解决plsql developer使用账号无法正常登录（前提：客户端等服务都已经安装好）
1.  查看一下oracle是否启动成功。在cmd 中输入 lsnrctl status 如出现类似以下4行说明启动成功。
 服务 "PLSExtProc" 包含 1 个例程。 
  例程 "PLSExtProc", 状态 UNKNOWN, 包含此服务的 1 个处理程序... 
 服务 "orcl" 包含 1 个例程。 
  例程 "orcl", 状态 READY, 包含此服务的 1 个处理程序... 
 服务 "orclXDB" 包含 1 个例程。 
  例程 "orcl", 状态 READY, 包含此服务的 1 个处理程序... 
 服务 "orcl_XPT" 包含 1 个例程。 
  例程 "orcl", 状态 READY, 包含此服务的 1 个处理程序...
2. 使用***管理员身份***登录cmd
3. 在cmd中执行sqlplus / as sysdba
4. 再执行alter user scott identified by aaa;
附：aaa为自己设置的密码，这个时候就可以在plsql developer中通过scott/aaa进行登录操作了。