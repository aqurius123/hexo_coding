---
title: String 聊源码解读
categories:
- 后端
tags: String
date: 
---
> 你真的了解String吗？之前一篇博客写jvm时，就觉得String可以单独拎出来写一篇博客，毕竟几乎所有的面试都是以String开始的，由此可以延伸出线程安全问题，jvm内存模型等问题。也以此告诫我们，作为一个技术开发人员，时刻需要关注底层的实现，保持刨根问底的好奇心的重要性！

这里提一下解读源码的思路：1.看其实现、继承->2.看其构造方法->3.看其重写的方法->4.了解其其他方法的实现

### 源码实现
1.以主流的jdk1.8来说，Spring 内部实际存储的结构为char数组，源码如下：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
    //...其他内容省略
}    
```
### 构造方法
我们先来看看它重要的4个构造方法：
```java
// String 为参数的构造方法
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
// char[] 为参数的构造方法
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
// StringBuffer 为参数的构造方法
public String(StringBuffer buffer) {
    synchronized(buffer) {
        this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
// StringBuilder 为参数的构造方法
public String(StringBuilder builder) {
    this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```
可以看到除了String 为参数的构造方法是直接赋值，其他三个方法都是调用Arrays.copyOf()方法复制一份等长的数据，并且StringBuffer考虑到线程安全的问题，使用了synchronized关键字。

Arrays.copyOf()实际上是调用了底层的实现(native本地方法，实际调用了C的方法库，对内存进行读写操作)：System.arraycopy(original, 0, copy, 0,Math.min(original.length, newLength)); 



### equals
String 重写了equals() 方法，源码如下：
```java
/* @see  #compareTo(String)
* @see  #equalsIgnoreCase(String)
*/
public boolean equals(Object anObject) {
    if (this == anObject) { // 对象引用相同直接返回true
        return true;
    }
    if (anObject instanceof String) { // 判断值是否为String类型
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            // 把两个值都转为char[] 数组对比
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 循环比对两个字符串的每一个字符
            while (n-- != 0) {
                // 如果有一个字符不相同就返回false
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```
还有一个和equals()比较类似的方法equalsIgnoreCase(),用于忽略字符串大小写后进行字符串比对！

compareTo()方法,用于比较两个字符串，返回int类型，源码：
```java
public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        // 获取到两个字符串长度最短的那个长度
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        // 对比每个字符
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                // 有字符不相等就返回差值（隐式转换 a为1 z为26）
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```
从源码可以看出，compareTo方法会循环对比所有的字符，当连个字符串中有任意一个字符串不相同时，就返回差值。当相等时返回0；（注：equals 和 compareTo只比较字符层面是否相等，不比较对象的引用是否一致）
例如下代码：
```java
String str1 = "java";
String str2 = "java";
String str3 = new String("java");
String str4 = new String("java");

System.out.println(str1==str2); // true
System.out.println(str1.equals(str2)); // true
System.out.println(str1.compareTo(str2)); // 0

System.out.println(str2==str3); // false
System.out.println(str2.equals(str3)); // true
System.out.println(str2.compareTo(str3)); // 0

System.out.println(str3==str4); //false
System.out.println(str3.equals(str4)); // true
System.out.println(str3.compareTo(str4)); // 0
```
可以看出，equals() 和 compareTo() 方法是等价的，唯一的不同是equals(Object),compareTo(String) 参数不同！

### 其他方法
- indexOf() 查询字符串首次出现的下标位置
- lastIndexOf() 查询字符串最后出现的下标位置
- contain() 查询字符串是否包含另一个字符串
- toLowerCase() 把字符串全部转为小写
- toUpperCase() 把字符串全部转换为大写
- length() 查询字符串长度 （数组查看长度size(),但是前台数组查看长度依然是length()，有时候前端写忘记了关键还不报错！）
- trim() 去掉首位空格
- replace() 替换字符串中某些字符
- split() 把字符串分割并返回字符串数组
- join() 把字符串数组转换为字符串

### 常见面试题
1.为什么String 要有final修饰？
2.String中StringBuilder 和StringBuffer 有什么区别？
3.String 的intern() 方法有什么含义？
4.String 类型在jvm 中是如何存储的？编译器对String 做了哪些优化？

接下来我们一个个看下这些问题的答案：
1.为了安全和高效的考虑，如果不是final的话，传参和内部指令调用时，它的值被改变了的话可能会引起不可预知的系统崩溃问题，且传参的时候需要重新拷贝一个新值，性能上会有一定损失！
2.StringBuilder 是非线程安全的，StringBuffer是线程安全的，但是考虑了线程安全就兼顾不了性能，在非并发的操作下我们选择StringBuilder来操作字符串的拼接。
3.intern() 方法是将字符串保存到常量池中。
```java
String s1 = "java";
String s2 = s1.intern();
String s3 = new String("java");
String s4 = s3.intern();

System.out.println(s1 == s2); // true
System.out.println(s1 == s3); //false
System.out.println(s3 == s4);// false s3在堆中 s4在常量池中
```
4.编译器堆代码进行了优化如下：
```java
String s1 = "ja" + "va";
String s2 = "java";
System.out.println(s1 == s2);//true
```
其中"ja" + "va"被直接编译成了"java".因此s1==s2才成立！

>小结：String的面试点基本就在== equals()和StringBuild和StringBuffer这里！还有要问就会问jvm 线程并发了。还是要多看源码，知其然知其所以然!


