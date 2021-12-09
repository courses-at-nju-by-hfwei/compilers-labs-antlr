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

另外，根据每个类型的值以及操作可以得到以下约束：

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

> 具体描述请参考许畅老师的实验讲义Project\_2.pdf的3.2.2内容

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

当然，你也可以选择Java已经提供的集合类来表示符号表，但这种方式并不推荐。

至于在符号表里应该填什么，这取决于你本身，只要觉得方便，可以向符号表中填入任何内容，下面给出一种示范

```java
// 使用散列表实现的符号表，链表解决hash冲突，散列表节点内容
public class HashNode {
    String name;
    Type type;
    HashNode next;
}
```

### 遍历语法树以及发现语义错误

> C--语言的特性请参考许畅老师的实验讲义Project\_2.pdf的3.1.1中的7个假设
>
> 语义错误类型请参考3.1.1中的18种错误类型

在定义好类型以及符号表之后，下一步需要做的就是通过语法树上处理上下文相关的内容并检测出可能出现的违背了C--语言特性的语义错误。

以下叙述中`walker` 的`walk()`以及`visitor` 的`visit()` 统称为访问

通过L2实验，相信你已经对Antlr的listener模式或visitor模式有了一个大概的了解，它们分别使用不同的机制来访问树节点

在上下文分析中，如何获取上下文的内容至关重要，我们无非就是希望在访问父节点的方法中得到的值能够传递到访问子节点的方法中，以及在访问完一个子节点后，能够以某种方式将得到的值再返回给访问父节点的方法使用。这样说可能会有点抽象，请看以下例子：

对于规则`extDef: specifier extDecList SEMI` 常用来声明一个全局变量

对于`int a;` 我们可以得到这样一棵语法树

```
Program (1)
  ExtDef (1)
    Specifier (1)
      TYPE: int
    ExtDecList (1)
      VarDec (1)
        ID: a
    SEMI
```

在访问树节点`ExtDef` 的时候，按照顺序我们之后会依次访问`Specifier` 以及`ExtDecList` ，在后面`VarDec` 中遇到符号`a`的时候，如果它不在符号表里，我们需要将其添加到符号表中。但是，光有符号名是不够的，还需要有它的类型，这个类型则是在访问`Specifier` 后得到的。所以，我们需要在访问完`Specifier` 过后，将创建的整数类型`Type` 对象的引用传递给父节点`ExtDef` ，然后再传递到接下来访问`ExtDecList` 的方法中，再按需传给更底层的节点。

对于`struct A{int a;};` 我们可以得到这样一棵树语法树

```
Program (1)
  ExtDef (1)
    Specifier (1)
      StructSpecifier (1)
        STRUCT
        OptTag (1)
          ID: A
        LC
        DefList (1)
          Def (1)
            Specifier (1)
              TYPE: int
            DecList (1)
              Dec (1)
                VarDec (1)
                  ID: a
            SEMI
        RC
    SEMI
```

遇到符号`A` 的时候，在添加进符号表之前，我们既需要得到它的类型对象的引用，也需要在它的类型对象中添加成员`a` ，即需要访问`StructSpecifier` 时新建一个结构体类型的对象，与`a` 一起添加到符号表中，后续访问完`DefList` 过后，再将`a` 添加到结构体类型对象的成员变量中。

以上只是最普通的一种值传递流程，或许会有更直接的方法来实现不同树之间的值传递。

下面我们将介绍几种Antlr4中使用的值传递机制供大家参考，以完成值传递流程。



## 实验说明

最本质的错误

本次实验代码量偏大，请大家保持良好的面向对象编码风格，系统地设计类的职责以及各个类之间的调用关系。

若发现本文档有误或者考虑不周全，你可以与助教联系。
