---
title: 托管网银
categories:
- 项目总结
tags: DVP账号余额(需求)
date: 2021-01-31
---

> 首先，这是一个生产txt文件并推送智慧系统的需求，是通过自动任务来实现的；

## 自动任务实现流程
1. 自定义biz和flow,实现业务功能；
2. 在autotask项目的services.xml配置文件里面配置上自己的自动任务信息；

## 自动任务需要操作的公共配置文件
1. autotask项目的settings.xml文件：用来配置本地和服务器上文件存放的地址；
2. autotask项目的services.xml文件：
   a. 用来配置操作文件的文件服务器的地址、ip、端口等配置信息；
   b. 用来配置自己定义的定时任务；

## DVP账号余额需求所用的知识点
1. 记录日志： 不管自动任务执行成功与否，一定要往数据库里面记录自动任务执行记录信息，方便后面日志跟踪；(重点)
2. 自定义组件；
3. 自己写sql时，没有用到相关sql组件，一定要注意防止sql注入；
4. 自己写分页sql,通过自定义集合然后进行sql传参，防止sql注入；
5. 循环将分页查询到数据写入到指定文件中；
6. 使用StringBuffer()作为文件写入的字符串操作对象；
7. 使用do while作为循环操作；
8. 在自定义组件中新增String fileName = "uploadFile"属性，并将最终文件名称赋值给fileName后放入到域对象中;

## 重要代码记录
connection = LianaDBAAccess.getConnection(CORDS);
int begin=0;    //初始化分页起始页
int num =1000;  //初始化每页条数
List resList = null;
int flag = 0;   //初始化循环次数
int count = 0;  //计数
do {
    List inputList = new ArrayList();
    inputList.clear();
    inputList.add(参数1);
    inputList.add(参数2);
    inputList.add((begin+num)+"");  //设置每页条数
    inputList.add(begin + "");  //设置起始页
    resList = LianaDBAAccess.queryMultipleResult(connection,sql, inputList, outSize);
    int size = 0;
    if (resList != null && resList.size() > 0){
        size = resList.size();
    }
    StringBuffer fileText = new StringBuffer();
    for (int j=0; j< size; j++) {
        count ++;
        List record = (List) resList.get(j);
        for (var k=0; k < record.size(); k++>){
            fileText.append(record.get(k)).append(分隔符);
        }
        fileText.append(换行符);
        if (count%500==0){//每当缓冲区达到500条数据时，一次性写入
            fileText.append(换行符);
            LianaFile.writeStringToFileAuto(fileName,fileText.toString(),ENCODING,true,true);
            fileText = new StringBuffer;
            count = 0;
        }
    }
    fileText.append(换行符);
    if (!"".equals(fileText.toString())) {
                    LianaFile.writeStringToFileAuto(fileName,fileText.toString(),ENCODING,true,true);

    }
    begin = begin + num;    //下一次分页从第几条数据开始写；
    flag++;
}while(flag < 1000>);

