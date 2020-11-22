---
title: RabbitMq的总结与应用
categories:
- 后端
tags: rabbitMq
date:
---
> 面试的时候关于消息队列的相关知识，备受面试官青睐，具体关于消息队列使用的好处及如何使用是怎样的呢？消息队列有很多种，具体我们来学习了解一下RabbitMq。

### 为什么使用RabbitMq呢？
1. 使用简单，功能强大。
2. 基于AMQP协议。
3. 社区活跃，文档完善。
4. 高并发性能好，这主要得益于Erlang语言。
5. Spring Boot默认已集成RabbitMQ。

### RabbitMq的基本机构，从左到右依次有：
1. 生产者：
2. 连接通道：
3. 消息队列服务进程(包含消息队列交换机和消息队列)：
4. 消费者：

### RabbitMq的工作原理：
**发送消息**
1. 生产者与消息队列服务进程Broker建立TCP连接，并建立连接通道；
2. 生产者将消息通过连接通道交给消息队列服务进程，由交换机进行转发；
3. 交换机将消息转发到指定的队列；

**接收消息**
1. 消费者与消息队列服务进程建立TCP连接，并建立连接通道；
2. 消费者监听指定的队列；
3. 当有消息到消息队列时，消息队列服务进程将消息推送给消费者；
4. 消费者接收消息；

### RabbitMq的使用场景
1. 异步请求处理：传统系统调用其他服务时，必须等到其他服务反馈信息才能给到用户进行响应；
2. 应用程序解耦：降低程序直接的耦合性，维护性更高；
3. 流量削峰：经常用于抢购、秒杀等业务；

#### 流量削峰的处理方案
> 架设消息队列，对流量进行限流操作

达到目的：
1. 可以控制前端请求数量
2. 防止因为瞬时流量激增，导致服务器宕机

#### 秒杀业务流程图
![秒杀流程图](https://7n.w3cschool.cn/attachments/image/20170428/1493366255432156.png "秒杀流程")
1. 浏览器端，最上层，会执行到一些JS代码
2. 站点层，这一层会访问后端数据，拼html页面返回给浏览器
3. 服务层，向上游屏蔽底层数据细节，提供数据访问
4. 数据层，最终的库存是存在这里的，mysql是一个典型（当然还有会缓存）
##### 秒杀业务优化
***浏览器端：***
方案1. 当用户点击查询或者抢购后，前端处理按钮置灰，禁止用户重复提交请求；
方案2. 限制几秒钟的时间内，用户只可以提交请求一次；

***后端处理：***
1. 做请求队列，每次放行有限的请求去操作数据库

### RabbitMq的工作模式
1. Work queues:工作队列模式，一个生产者对应2个消费者，消息队列服务进程通过轮询的方式将消息推送给消费者；
![工作队列模式](http://123.57.91.223:8090/upload/2020/10/2-67789ad575e14293ad60b3aee8ac8a38-thumbnail.png)
>  结果：
>> 1. 一条消息只会被一个消费者消费；
>> 2. 消息队列采用轮询的方式将消息推送给消费者；
>> 3. 消费者在处理完某条消息后，才会收到下一条消息；

2. Publish/Subscribe：发布订阅模式，多个消费者对应着多个消息队列，各自监听自己对应的消息队列；
![发布订阅模式](http://123.57.91.223:8090/upload/2020/10/Publish-Subscribe-98541c3a75044c40aa1c18214b996a9b-thumbnail.png)
>  工作模式：
>> 1. 每个消费者监听自己的队列。
>> 2. 生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息;
3. Routing：路由模式
![路由模式](http://123.57.91.223:8090/upload/2020/10/routing-e1fb31d4e2e2472d9dda160d14b1c6fc-thumbnail.png)
>  工作模式：
>> 1. 每个消费者监听自己的队列,并且设置routingKey。
>> 2. 生产者将消息发给交换机，由交换机根据routingkey来转发消息到指定的队列。
4. Topic：通配符模式，可以指定处理某一类的消息；
5. Header 
6. RPC

### Work Queue和Publish/Subscribe的比较：
> 不同点：
1. Work Queue不需要手动操作交换机(底层默认有交换机)，但是Publish/Subscribe则需要手动创建交换机、绑定交换机

> 相同点：-----***但是这个地方是如何进行测试的呢？？？？***
1. 两者实现的发布/订阅的效果是一样的，多个消费端监听同一个队列不会重复消费消息。

**推荐使用Publish/Subscribe，因为功能更加强大，且可以指定自己专用的交换机**

### Publish/Subcribe和Routing的区别：
1. 基本操作都相同，唯一不同的是在声明将交换机和队列绑定的时候，有一个参数指定routingKey而已；
2. 在发送消息的时候绑定routingKey;





## 总结
### 一、关于RabbitMq的使用伪代码步骤如下：
> 生产者
>> 1. 定义队列名称和交换机名称(**注意用static和final关键字修饰**)；
>> 2. 定义连接工厂对象；
>> 3. 设置工厂对象属性(ip, 端口，用户名， 密码， 虚拟机服务路径)；
>> 4. 获取连接对象；
>> 5. 获取连接通道对象；
>> 6. 声明交换机对象(对于发布订阅模式才有的步骤);---channel.exchangeDeclare()
>> 7. 声明队列；---channel.queueDeclare()
>> 8. 将交换机绑定队列；-----channel.queueBind()
>> 9. 发送消息------channel.basicPublish()
>> 10. 关闭对象，释放资源(***这个步骤不能忘了***)

> 消费者
>> 1. 定义队列名称(***注意：此时并不需要定义交换机名称***)；
>> 2. 定义连接工厂对象；
>> 3. 设置工厂对象属性(ip, 端口，用户名， 密码， 虚拟机服务路径)；
>> 4. 获取连接对象；
>> 5. 获取连接通道对象；
>> 7. 声明队列；---channel.queueDeclare()
>> 9. 接收消息------new DefaultConsumer(channel)，重写handleDelivery方法
>> 10. 监听指定队列------channel.basicConsume()
>> 11. 关闭对象，释放资源(***这个步骤不能忘了***)

### 二、如何保证消息不丢失：
使用场景：对生产者和消费者进行集群搭建，以达到高可用；
> 问题1：
>> 1. 当消息到达消息队列并保存在内存中，如果消息在没有被消费者消费之前，消息队列宕机了怎么办？
>>> 方案： 可以考虑持久化操作，将队列和消息都进行持久化操作，写入磁盘中；

**队列的持久化操作:**
1. 通过声明队列的Api，设置参数持久化；

**消息的持久化操作：**
1. 在发送消息的时候通过Api，设置消息属性参数为持久化；

> 问题2
>> 1. 当rabbitmq未将消息写入到磁盘时，消息队列宕机了，怎么办？
>>> 方案：此时需要保证生产者发送的消息，rabbitmq会一定成功的进行队列与消息持久化，否则进行重发；

### 三、针对生产者投递消息给消息队列时，保证数据不会丢失，RabbitMq提供了两种解决机制：
1. 重量级的事务机制：极度影响性能，**不推荐使用**；
2. confirm机制：confirm模式需要基于channel进行设置, 一旦某条消息被投递到队列之后,消息队列就会发送一个确认信息给生产者,如果队列与消息是可持久化的, 那么确认消息会等到消息成功写入到磁盘之后发出；
	confirm的性能高,主要得益于它是异步的.生产者在将第一条消息发出之后等待确认消息的同时也可以继续发送后续的消息.当确认消息到达之后,就可以通过回调方法处理这条确认消息. 如果MQ服务宕机了,则会返回nack消息. 生产者同样在回调方法中进行后续处理，核心代码如下：
    ~~~
    try {
            // 开启confirm模式
            channel.confirmSelect();
            //设置监听器
            channel.addConfirmListener(new ConfirmListener() {
                public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                    //todo--删除之前临时存储空间的消息
                    System.out.println("ack: deliveryTag" + deliveryTag + ",multiple: " + multiple);
                }
                public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                    //todo--从临时存储空间取出消息
                    System.out.println("nack: deliveryTag: " + deliveryTag + ", multiple: " + multiple);
                }
            });
            //循环发送消息
            for (int i = 0; i < 100; i++) {
                channel.basicPublish(exchangeName, routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, messageByte);
            }
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            channel.close();
            connection.close();
        }



### 四、当消费者宕机了怎么办？
当消费者成功消费消息后，手动给消息队列返回一个应答信息(ACK),注意，虽然说ACK是自动应答，但是可能会出现其他服务还没来的急消费相关信息而导致消息被删除的场景(例如：订单、库存和物流系统之间的关系，所以最好通过参数设置手动应答，由程序来决定什么时候决定什么时候可以让消息队列删除该条消息)， 核心代码如下：
~~~
DefaultConsumer consumer = new DefaultConsumer(finalChannel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {

        String value = new String(body,"utf-8");
        try{
            //仓储服务业务
            //调用物流系统
            if(调用成功)｛
            //通知消息队列删除此消息
            finalChannel.basicAck(envelope.getDeliveryTag(),false);
            ｝else{
                //重新调用
            }
        }catch（Exception e）{
            //特定异常特定处理
        }
    }
};
//将第二个参数设置为false，则表明开启手动应答
channel.basicConsume(QUEUE,false,consumer);
~~~

### 五、ACK的工作原理
生产者将生成好的消息(包含有唯一标志deliveryTag)交给消息队列进行管理，当消息队列将消息推送给消费者成功消费后，消费者手动ack的方法，将该消息的deliverTag返回给消息队列，消息队列最终才会删除当前已被消费的消息；

### 六、如果一旦消费者宕机了, 那么这个订单消息不就卡在消息队列了么?
对于当前的架构设计. 库存服务是以集群方式部署. 会存在多个库存服务的实例. 对于rabbitMQ生产者来说, 如果一旦发现某个库存服务(消费者)宕机了. 那么就会将这个订单消息发送给其他的库存服务实例(消费者)去使用这条消息；

### 七、rabbitMQ是如何感知到消费者宕机的?
消费者实例已经注册到了rabbitMQ, 所以rabbitMQ与消费者实例是存在联系的,当消费者实例宕机,rabbitMQ必然会知道。

### 八、当rabbitMQ感知到某一个消费者实例宕机,它是如何进行消息重发的?
当设置了手动ACK。在代码中可以进行try/catch操作，一旦发现任何异常，则进行消息重发，核心代码如下：
~~~
DefaultConsumer consumer = new DefaultConsumer(finalChannel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {

        String value = new String(body,"utf-8");
        try{
            //调用发货
            //通知消息队列删除此消息
            finalChannel.basicAck(envelope.getDeliveryTag(),false);
        }
        catch(Exception e){
            //异常处理
            //第一个参数: 消息表示信息
            //第二个参数:通知MQ服务当前消费者实例没有处理成功,让MQ服务将这个消息重新投递给其他消费者实例
            //如果设置为了false,会导致就算MQ服务知道当前消费者实例没有处理成功, 但是依旧会删除这个消息.
            channel.basicNack(envelope.getDeliveryTag(),true)
        }
    }
};
channel.basicConsume(QUEUE,false,consumer);
~~~

### 九、如果其他库存实例调用完了此订单消息,但是刚才的库存服务又重新启动了,那因为它刚才已经接收到了消息,它又去根据这个订单消息去调用物流系统,但是其他库存服务已经用完了这个订单消息, 怎么办?

此问题无需考虑. 因为库存服务只是一个消费者, 它只会去持续监听消息队列,拿消息进行使用.而不会对消息进行存储.所以该问题不会发生.

### 十、基于ack机制,结合高并发场景会出现什么问题?

对于当前的操作, 每一个channel都会存在若干的unack消息(未确认消息). 比方说, rabbitMQ正在发送的消息 、 消费者实例接收到消息之后但没有处理完 、 执行了ack但是因为ack是异步的也不会马上变为ack信息 、 开始批量ack延迟时间会更长. 对于这些场景,都会存在unack的消息. 此时如果rabbitMQ无限制的过多过快的向消费者实例发送消息,就会导致庞大的unack消息积压在消费者实例的内存中,如果继续保持发与积压的状态,最终会导致消费者实例的OOM（out of memory内存溢出）！！！

### 十一、保证消息有序性
只所以可能会出现消息出现不同顺序的被消费，是因为有多个消费者共同监听着一个队列；可以设置只有一个消费者监听该队列，这样就能保证这些消息只推送给一个消费者，这样就能保证了消息被有序消费了；

### 十二、保证消息幂等性
> 幂等性：对于相同的一次请求操作，不管操作多少次，得到的结果都是相同的。

> 场景：
	在开启消费者手动ack的情况下, 库存服务的消费者已经接收到了消息,并调用物流系统并成功更改状态进行发送, 但是在ack返回消息的时候, 这个消费者服务可能突然宕机,  因为消费者ack未返回, 所以会导致这个消息一直在MQ中, 当消费者重新启动就会又接收这个消息进行发货, 这样的话,就会导致相同的货品发货了两次!!!
> 解决方案：
>> 1. 状态判断：在消费者接收消息执行之前, 先根据消息的标识信息查询DB,判断状态, 如果为已发货/待发货状态的话,则直接ack返回MQ,删除这条消息;
>> 2. 乐观锁



