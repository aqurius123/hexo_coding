---
title: java控制结构
categories:
- 后端
tags: 控制结构
date: 2021-03-07
---

> java控制结构包括：顺序控制结构、分支控制结构和循环控制结构

## 顺序控制结构
java语言从上至下、从左到右依次执行

## 分支控制结构
1. if条件分支
2. switch...case语句：
   
   ##注意事项：
   
   1. switch 语句中的变量类型可以是： byte、short、int 或者 char。从 Java SE 7 开始，switch 支持字符串 String 类型了，同时 case 标签必须为字符串常量或字面量。
   2. 当变量的值与 case 语句的值相等时，那么 case 语句之后的语句开始执行，直到 break 语句出现才会跳出 switch 语句。
   3. 当遇到 break 语句时，switch 语句终止。程序跳转到 switch 语句后面的语句执行。case 语句不必须要包含 break 语句。如果没有 break 语句出现，程序会继续执行下一条 case 语句，直到出现 break 语句。
   4. switch 语句可以包含一个 default 分支，该分支一般是 switch 语句的最后一个分支（可以在任何位置，但建议在最后一个）。default 在没有 case 语句的值和变量值相等的时候执行。default 分支不需要 break 语句。

   ### Switch...case特别注意
   1. 每个case分支都必须加上break,**否则会出现语句穿透现象---也就是说从某个分支匹配开始，后面的分支都会执行**
   2. defalut分支是只有当其他分支都不匹配的情况下才会默认执行的，一般都放在最后一个分支；

   ### 基本案例
~~~
   public class SwitchCaseTest {
    public static void main(String[] args) {
        String condition = "3";
        switch (condition){
            case "1":
                System.out.println("条件为1时执行");
            case "2":
                System.out.println("222222");
                break;
            default:
                System.out.println("可以理解为当没有条件满足时最终默认执行");
        }
    }
}
~~~

### Swithc...case和If...else的比较
1. 灵活性：if...else语句条件判断相对更加灵活；
2. 效率：
   ① 在分支比较多的情况下优先考虑switch case，不仅考虑到效率，也应该综合代码可读性，可维护性等。
② 选择使用哪个表达式之前应该评估每个条件出现的可能性，如果if else的前面两三个条件就可以覆盖到绝大部分情况，其实if else的效率并不会比switch case低。
③ 这两种表达式的不同在效率上对整体代码的执行时间影响很小，应该把代码的可读性，可维护性等考虑因素放在效率之前。
————————————————
原文链接：https://blog.csdn.net/weixin_43862101/article/details/84859437
3. 可读性和可维护性
**总的来说：视具体业务场景来决定具体使用if...else还是switch...case**

## 循环控制结构
- for循环

- while循环

- do while循环

### 循环会配合break、continue、return的使用 
