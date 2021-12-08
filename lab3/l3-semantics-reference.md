---
description: TODO
---

# L3-Semantics-Reference

## 实验介绍

在完成了L1词法分析以及L2语法分析后，我们将会从上下文无关分析阶段进入到上下文相关分析阶段。顾名思义，在之前的阶段中，我们并没有处理与输入代码的上下文相关的内容，例如在函数体中给一个没有声明的变量赋值，将一个INT类型的变量赋值为结构体，传入函数的参数与函数定义时的参数不同...等等。这些与上下文相关的内容就会在本次实验中进行处理。

本次实验要求你识别代码中的语义错误并打印，如果没有错误就不用输出任何内容。

## 实验指导



## 实验步骤

### 设计类型

> 类型：一组值，以及在这组值上的一系列操作。当我们在类型上面尝试去执行其不支持的操作时，类型错误就会产生。

给定一个符号`a` ，在Cmm语言中，它可以是`INT` 和`FLOAT` 这样的基本类型，也可以对应由基本类型构造出来的`STRUCTURE` 以及`ARRAY` ，当我们在`a` 的后面添加括号时，它又可以被理解成一个`FUNCTION` 。

你可以创建多个类来表示上述的类型，并继承一个抽象父类`TYPE` ，并且声明一个成员变量来表明其属于哪个类型，类型可以使用枚举类来实现。

```java
public enum Kind {
    INT,
    FLOAT,
    ARRAY,
    STRUCTURE,
    FUNCTION
}
```

另外，根据每个类型的值以及操作：

* 基本类型的类无需存储其他内容
* `ARRAY` 类需要存储其element类型以及element数量
* `STRUCTURE` 类需要存储其每个成员类型
* `FUNCTION` 类需要存储其返回类型以及每个参数的类型

当然，你也可以根据你的理解，合理的实现不同的类型类。

对于`FUNCTION` 的参数存储以及`STRUCTURE` 的成员存储，你可以选择链表或者数组列表这样的的数据结构实现，使用链表结构的代码实现如下

```java
public class Function extends Type {

    private Type returnType;
    private FieldList paramList;
    ...
}
public class Structure extends Type {

    private FieldList memberList;
    ...
}
// FieldList并不是一个Type，仅用来存储函数的参数以及结构体的成员
public class FieldList {

    private String name;
    private Type type;
    private FieldList next;
    ...
}
```

有了以上示范，`ARRAY` 类以及基本类型类则很好实现了

以上代码仅供参考，实际编写代码时请将不同类放到不同java文件中，并灵活根据需要修改其内容

### 设计符号表

> 具体描述可以参考许畅老师的实验讲义Project\_2.pdf的3.2.2内容

设计好类型后，下一步你应该思考如何存储一个符号的类型信息，以及给定一个符号名，如何得到它的类型信息，这就需要符号表来帮忙。

符号表上的操作包括填表和查表两种，在定义一个新的符号的时候，你需要往符号表里添加这个符号名以及它对应的信息；在使用一个符号的时候，你需要判断符号表中是否存在这个符号的信息并且从符号表中得到这个符号的所有信息。

符号表的组织方式有多种，你可以将程序中出现的所有符号组织成一张全局表，也可以将不同种类的符号组织成不同的表等等。

至于符号表该用哪种数据结构，你可以按照你的需求自由实现，包括但不限于

* 线性链表，添加效率极高，查找和删除效率较低
* 平衡二叉树，较高的查找，添加，删除效率，但实现难度较高
* 散列表，查找，添加，删除效率都极高，但是需要解决哈希冲突的问题，是符号表的实现中最常被采用的数据结构
* ...

对于散列表，下面给出了一个不错的hash函数选择，`HASH_TABLE_SIZE` 描述了散列表的大小，

```java
private int getHashIndex(String name) {
    int val = 0, i;
    for (char c : name.toCharArray()) {
        val = (val << 2) + (int) c;
        // HASH_TABLE_SIZE描述了符号表的大小
        if ((i = (val & ~HASH_TABLE_SIZE)) != 0) {
            val = (val ^ (i >> 12)) & HASH_TABLE_SIZE;
        }
    }
    return val;
}
```

当然，你也可以选择Java已经提供的集合类来实现符号表，但这种方式并不推荐。

至于在符号表里应该填什么，这取决于你本身，只要觉得方便，可以向符号表中填入任何内容，下面给出一种示范

```java
// 使用散列表实现的符号表，链表解决hash冲突，散列表节点内容
public class HashNode {
    String name;
    Type type;
    HashNode next;
}
```

### 语义错误提示

在完成L2实验后，相信你已经对Antlr的listener模式或visitor模式有了一个大概的了解

## 实验说明

本次实验代码量偏大，请大家保持良好的面向对象编码风格，系统地设计类的职责以及各个类之间的调用关系。

若发现本文档有误或者考虑不周全，你可以与助教联系。
