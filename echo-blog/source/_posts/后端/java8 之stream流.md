---
title: jdk1.8新特性之stream流
categories:
- 后端
tags: stream
date: 
---
>在jdk1.5的时候，我们需要掌握枚举：反射、注解、泛型。现在java14都出来了
jdk1.8的新特性：函数式接口、链式编程、stream流、lambda表达式 都掌握的怎么样了？

**本篇将着重说明 Stream 流的用法**
### 面试题：
按条件筛选用户，请你只用一行代码完成！
* 1、id 为偶数
* 2、年龄大于24
* 3、用户名大写
* 4、用户名倒排序
* 5、输出一个用户

代码(User 实体类省略)：
```java
package stream;

import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
public class StreamDemo {
    public static void main(String[] args) {
        User user1 = new User(1,"jim",23,"北京");
        User user2 = new User(2,"tom",24,"武汉");
        User user3 = new User(3,"echo",25,"深圳");
        User user4 = new User(4,"jerry",26,"上海");
        User user5 = new User(5,"bob",27,"北京");
        //数据库、集合 ： 存数据的
        // Stream：计算和处理数据交给 Stream
        List<User> users = Arrays.asList(user1, user2, user3, user4, user5);
        users.stream()
                .filter(u->{return u.getId()%2 == 0;})
                .filter(u->{return u.getAge() > 24;})
                .map(u->{return u.getName().toUpperCase();})
                .sorted((u1,u2)->{return u2.compareTo(u1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```
>接下来我们深入看看stream流中都有些什么？

### 创建流：
```java
//1.创建一个具体字符串流
Stream<String> stream1 = Stream.of("A", "B", "C", "D");
//2.创建一个Stream流Builder<Object>对象
Stream.Builder<Object> builder = Stream.builder();
//3.创建一个空的String 流
Stream<String> empty = Stream.empty();
//4.合并两个流
Stream<String> concat = Stream.concat(stream1, empty);
//5.用迭代器创建无限流
Stream<Integer> iterate = Stream.iterate(1, x -> x + 1);
//6.生成 无限流
Stream<Double> generate = Stream.generate(() -> Math.random());
//7.collection的串行流  和并行流
List<String> list = Arrays.asList("1","2","3","4");
Stream<String> stream2 = list.stream();
Stream<String> stream3 = list.parallelStream();
//8.Arrays.stream创建一个数组流
IntStream stream = Arrays.stream(new int[]{1, 2, 3, 4});
//9.通过文件生成字符串流
Stream<String> stream = Files.lines(Paths.get("text.txt"), Charset.defaultCharset());
```
### 流的使用：
看的流的使用 也就是看users.stream()能点出来哪些东西,因为太多，这里就举例说明常用的几种：
#### 1.filter过滤：
```java
// 筛选出>3的数据
List<Integer> list = Arrays.asList(1,2,3,4);
list.stream().filter((i)->{return i > 3;}).forEach(System.out::println);
// 输出4
```
#### 2.limit限流
```java
// 获取未来7天的日期(顺便看看iterate 和 generate的用法)
        Stream.iterate(LocalDate.now(), date -> date.plusDays(1)).limit(7).forEach(date-> {
            System.out.print(date+",");
        });
        // 输出 2020-03-12,2020-03-13,2020-03-14,2020-03-15,2020-03-16,2020-03-17,2020-03-18,
        AtomicInteger a = new AtomicInteger(0);
        // 截取前三个随机数 并打印
        // 写在这里的 时候就想换行输出下 加个计数器判断下
        // 至于为什么用AtomicInteger计数而不是int,我猜是因为设计者考虑到并发情况下线程安全的问题 
        // 因为“ Variable used in lambda expression should be final or effectively final”
        // AtomicInteger 在另一篇博客【并发编程之美-JUC]中有提到过 
        Stream.generate(()->Math.random()).limit(3).forEach(d->{
            a.getAndIncrement();
            if(a.get() > 1){
                System.out.print(d+",");
            }else{
                System.out.print("\n"+d+",");
            }
        });
        //输出0.8662508892898771,0.26661993344781665,0.2584450405261183,
```
#### 3.skip 跳出
```java
//skip（n）去掉前n个元素的流
List<String> list = Arrays.asList("1","2","3","4");
//若流中元素不足n个，则返回一个空，与limit（n）互补。
list.stream().skip(3).forEach(System.out::print);
//输出4
```
#### 4.sorted排序
```java
//倒序排列
List<String> list = Arrays.asList("1","3","5","2","4");
list.stream().sorted((o1, o2)->{return o2.compareTo(o1);}).forEach(System.out::print);
//输出54321
```
#### 5.distinct筛选
```java
//去除重复数据
List<String> list = Arrays.asList("1","2","3","1","2");
list.stream().distinct().forEach(System.out::print);
//输出123
```
#### 6.映射
```java
// 流式计算将实体中某两个属性对应组装成key value的格式返回
// 项目中一般读取数据字典 根据code 返回前台数据使用
Map<Integer, String> collect = users.stream()
.collect(Collectors.toMap(User::getId, User::getName));
System.out.println(collect.toString());
// 输出{1=jim, 2=tom, 3=echo, 4=jerry, 5=bob}

//将user按adress 分组
Map<String, List<User>> addressMap = users.stream()
.collect(Collectors.groupingBy(User::getAddress));
System.out.println(addressMap.toString());
//输出：{上海=[stream.User@52cc8049], 武汉=[stream.User@5b6f7412],
//深圳=[stream.User@27973e9b], 北京=[stream.User@312b1dae, stream.User@7530d0a]}
```
> 小结：以上就是stream 流的常见用法 至于规约 查找 匹配都用的都很少，暂且不再深入 ，关于lambda表达式和函数式接口后面会写,还有一个很好玩的类Optional，后面也来写写看，奥利给！！!
