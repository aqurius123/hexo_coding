---
title: java8 新特性之optional
categories:
- 后端
tags: optional
date:
---
> 拒绝非空判断，我们一起来折腾下`java8`的新特性`optional`类 吧；

### 概念

为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Java 8类库 引入了一个同名的Optional类。实际上是个Optional容器类：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测，使你从繁琐的非空判断中解脱出来，写出更加优雅的代码！

### 快速上手

废话不多说，看源码一个个方法捋一遍！

1. 私有方法和构造器

```java
// 私有静态常量，一个空的Optional对象
private static final Optional对象<?> EMPTY = new Optional<>();
// 私有泛型value 成员变量
private final T value;
// 私有无参构造
private Optional() {
    this.value = null;
}
// 私有有参构造
private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}
```

从上面可以看出我们不能通过构造器创建一个Optional 对象，那么我们就来结合代码例子看看可以通过哪些方法创建Optional对象。

2. 其他方法

```java
// 创建一个包装对象值为空的Optional对象
Optional<String> optional1 = Optional.empty();
// 创建包装对象值非空的Optional对象
Optional<String> optional2 = Optional.of("hello,optional");
// 创建包装对象值允许为空的Optional对象
Optional<String> optional3 = Optional.ofNullable(null);
```

3. get

```java
System.out.println(optional2.get());
// 输出hello，optional
// 可以看到get方法获取optional对象的实际值。但是optional对象值为null，会抛出NoSuchElementException异常

// get源码如下：
// public T get() {
//     if (value == null) {
//         throw new NoSuchElementException("No value present");
//     }
//     return value;
// }
```

4. isPresent

```java
System.out.println(optional2.isPresent());
//输出true
//isPresent()方法用于判断包装对象的值是否非空

// public boolean isPresent() {
//     return value != null;
// }
```

5. ifPresent

```java
UserInfo userInfo = new UserInfo(1,"小明","深圳");
Optional.ofNullable(userInfo).ifPresent(u ->  System.out.println("The user name is : " + u.getName()));
// 输出The user name is 小明

// public void ifPresent(Consumer<? super T> consumer) {
//     if (value != null)
//     consumer.accept(value);
// }
// 从源码可以看出ifPresent 方法接受一个Consumer 函数，内部做了null值检查，调用前无需担心NPE问题
```

6. filter

```java
System.out.println(Optional.ofNullable(userInfo).filter(u ->u.getId() > 1).isPresent());
// false

// public Optional<T> filter(Predicate<? super T> predicate) {
//     Objects.requireNonNull(predicate);
//     if (!isPresent())
//         return this;
//     else
//         return predicate.test(value) ? this : empty();
// }
// 可以看到filter 方法接受一个断言型函数，过滤出我们想要的数据 当为空是返回一个 空对象
```

7. map

```java
System.out.println(Optional.ofNullable(userInfo).map(u -> Optional.u.getName()).get());
// 输出 小明

// public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
//     Objects.requireNonNull(mapper);
//     if (!isPresent())
//         return empty();
//     else {
//         return Optional.ofNullable(mapper.apply(value));
//     }
// }
// 可以看到map 方法接受一个函数式接口对象 首先会判断非空 然后返回一个Optional 对象
```

8. flatMap

```java
Optional<Integer> integer = Optional.ofNullable(userInfo).flatMap(u -> Optional.ofNullable(u.getId()));
System.out.println(integer.get());
// 输出 1

// public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
//     Objects.requireNonNull(mapper);
//     if (!isPresent())
//         return empty();
//     else {
//         return Objects.requireNonNull(mapper.apply(value));
//     }
// }
// 可以看到flatMap 接受一个函数式接口的参数，但和 map 方法，函数式接口类实例有两个类型一个是泛型T ，一个是Optional<U> 对象
// 而map 方法是一个U类型对象 但是可以看到都作了requireNonNull 判断
```

9. orElse()

```java
userInfo = null;
String uName = Optional.ofNullable(userInfo).map(u -> u.getName()).orElse("Unknown name");
System.out.println(uName);
// 输出 Unknown name

// public T orElse(T other) {
//     return value != null ? value : other;
// }
// 可以看到 orElse 方法中实际就是一个三目运算 ,不为null就是当前值, 为null就是输入的值
```

10. orElseGet

```java
userInfo = null;
String uName = Optional.ofNullable(userInfo).map(u -> u.getName()).orElseGet(() -> "unknown name");
System.out.println(uName);
// 输出 unknown name

// orElse
//public T orElseGet(Supplier<? extends T> other) {
//	 return value != null ? value : other.get();
//}
// 可以看到 orElseGet 和 orElse接受的参不同，一个是泛型，一个提供者接口函数对象
// 我们可以在对应的函数中写自己的逻辑，返回get到的实际值
```

11. orElseThrow

```java
UserInfo userInfo1 = new UserInfo(2,"小花",null);
String exception1 = Optional.ofNullable(userInfo1).map(u ->
       u.getName()).orElseThrow(() -> new Exception("没有获取到名字"));
System.out.println(exception1);
// 输出 小花

String exception2 = Optional.ofNullable(userInfo1).map(u ->
       u.getPassword()).orElseThrow(() -> new Exception("没有获取到密码"));
System.out.println(exception2);
// 这个报了个我们自定义的异常“Exception in thread "main" java.lang.Exception: 没有获取到密码”

//public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) //throws X {
//    if (value != null) {
//        return value;
//    } else {
//        throw exceptionSupplier.get();
//    }
//}
// 可以看到 orElseThrow 在有值得是后返回我们逻辑处理的值  没有值得时候抛出了我们自定义的异常
```

12. equals，hacode，toString 最后这三个都重写了Object的方法，其实没什么好说的，提一下：

    equals使用Objects.equals(value, other.value) ，那意味着我们写equals判断时也应该这么写！

    hashCode使用Objects.hashCode(value)

    toString使用了三目运算和String.format("Optional[%s]", value) 占位符，意味着我们有些打印日志可以这么写。这就是面向源码学习，多借鉴借鉴还是很不错滴！

> 小结：至此，我们已经从头到尾看了遍optional底层的实现了！ 然后平时过滤啥的知道用filter，改变值用map，判空用isPresent, if else 判断啥的用orElse 或者 orElseGet了吧，那就动手实践起来，话说最近在看一本书《重构改善既有代码的设计》,很不错推荐一哈！