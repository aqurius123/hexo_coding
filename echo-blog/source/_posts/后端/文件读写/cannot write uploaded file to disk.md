---
title: 
categories:
- 后端
tags: 
date:
---

> 使用场景：操作文件时，电脑磁盘无法进行写入操作,直接报cannot write uploaded file to disk；

最终原因：同名文件被占用操作问题；

解决方案：对文件的读写路径做机构和当前时间戳的拼接(可以解决问题，但是个人觉得不是很完美的解决方案);

反思：同样是debug断点走代码，为什么龙哥就知道在delete()这个地方有问题，程序并没有真正被删除？而自己却始终将错误确定在上传file.transferTo()上面呢？