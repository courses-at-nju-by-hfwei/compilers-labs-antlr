---
description: Antlr语义分析实验指导
---

# L3-Semantics-Reference

## 实验介绍

在完成了L1词法分析以及L2语法分析后，我们将会从上下文无关分析阶段进入到上下文相关分析阶段。顾名思义，在之前的阶段中，我们并没有处理与输入代码的上下文相关的内容，例如在函数体中给一个没有声明的变量赋值，将一个INT类型的变量赋值为结构体，调用函数时传入的参数与函数定义的参数不相同等等。

本次实验的关键就在于如何处理代码中上下文相关的内容，并且检查各种行为都是安全合法，即语义正确的。如果检测到了代码中的语义错误，就需要将其打印出来，没有错误就不用输出任何内容。

## 实验步骤

### 设计类型

> 类型：一组值，以及在这组值上的一系列操作。当我们在类型上面尝试去执行其不支持的操作时，类型错误就会产生。

为了满足类型检查的需要，我们就需要将Cmm语言中定义的类型用特定的数据结构实现出来

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
* `STRUCTURE` 类需要存储其每个成员类型，对于非匿名结构体还需要存其结构体名
* `FUNCTION` 类需要存储其返回类型以及每个参数的类型

当然，你也可以根据你的理解，合理的实现不同的类型类。

对于`FUNCTION` 的参数存储以及`STRUCTURE` 的成员存储，你可以选择链表或者数组列表这样的的数据结构实现，使用链表结构的代码实现如下

```java
public class Function extends Type {

    private Type returnType;
    private Field paramListHead;
    ...
}
public class Structure extends Type {
    
    private String name;
    private Field memberListHead;
    ...
}
// Field并不是一个Type，仅用来存储函数的参数以及结构体的成员
public class Field {

    private String name;
    private Type type;
    private Field next;
    ...
}
```

有了以上示范，`ARRAY` 类以及基本类型类则很好实现了

以上代码仅供参考，实际编写代码时请将不同类放到不同java文件中，并灵活根据需要修改其内容。

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
// 使用散列表实现的符号表，散列表节点内容如下
public class HashNode {
    String name;
    Type type;
    // 链表解决hash冲突
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

规则`extDef: specifier extDecList SEMI | specifier SEMI;` 常用来声明一个全局变量

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

在访问树节点`ExtDef` 的时候，按照顺序我们之后会依次访问`Specifier` 以及`ExtDecList` ，在后面`VarDec` 中遇到符号`a`的时候，如果它不在符号表里，我们需要将其添加到符号表中。但是，光有符号名是不够的，还需要有它的类型，这个类型则是在访问`Specifier` 后得到的。所以，我们需要在访问完`Specifier` 过后，将创建的整数类型对象的引用传递回访问父节点`ExtDef` 的方法中，然后再传递到接下来访问`ExtDecList` 的方法中，再按需传给更底层的节点。

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

在收集信息和计算数值的时候，最方便和快捷的方法就是传递参数和使用方法返回值，而不是使用一些类成员变量和所谓的全局变量。但问题在于Antlr自动生成的listener监听器方法是不支持自定义返回值和参数的，同样，Antlr生成的visitor访问器方法也不支持自定义参数。

下面我们将介绍几种Antlr4中使用的值传递机制供大家参考，以完成值传递流程。

> 具体例子可以参考_The Definitive ANTLR 4 Reference (2nd Edition) Chapter 7.5_

#### 使用Visitor遍历语法分析树

在使用Antlr生成对应的visitor代码后，可以看到生成的泛型类`CmmParserBaseVisitor` ，并且其中所有的函数返回值都是由泛型定义的，默认都是`null` 。要想实现自定义的返回值类型，我们可以新建一个visitor类继承该泛型类，并且在定义类的时候初始化父类的`T` ，如下

```java
public class CmmSemanticVisitor extends CmmParserBaseVisitor<XXX> {
    @Override
    public XXX visit(ParseTree tree) {
        ...
    }

    @Override
    public XXX visitTerminal(TerminalNode node) {
        ...
    }
    @Override
    public XXX visitProgram(CmmParser.ProgramContext ctx) {
        ...
    }

    @Override
    public XXX visitExtDef(CmmParser.ExtDefContext ctx) {
        ...
    }
}
```

在<>内部，我们就可以指定具体类替换掉`T` ，这样我们子类的所有方法返回值就会变为自定义的具体类，之后在方法中实现具体逻辑就好。

不同的规则树需要返回的类型其实是不同的，如何统一为一个类型需要大家注意。

#### 使用栈来模拟返回值

Antlr生成的监听器方法是没有返回值的（`void` 类型），为了向更高层的调用者返回值，我们可以把目前监听器方法中的结果保存在当前监听器的成员变量中，函数调用的过程无非就是压栈和出栈的过程，那么我们就可以使用一个栈结构来临时推入返回值，在更高层的监听器方法中推出。

这保证了事件方法在所有的监听器事件之间的执行顺序是正确的，但是不够优雅， 前一种带返回值的访问器较为规范，但是又需要我们手动触发对树节点的访问。

```java
public class CmmSemanticListener extends CmmParserBaseListener{
    // 不推荐使用java.util.Stack类
    Deque<XXX> stack = new ArrayDeque<>();
    ...
    // specifier: structSpecifier| TYPE;
    @Override public void exitSpecifier(CmmParser.SpecifierContext ctx) {
        ...
        // 得到这个specifier对应的具体类型对象后，压栈
        stack.push(type)
        ...
    }
    ...
    // paramDec: specifier varDec;
    @Override public void exitParamDec(CmmParser.ParamDecContext ctx) {
        ...
        // 得到访问完子树specifier后得到的类型
        XXX type = stack.pop();
        // 给vardec中得到的ID或者数组中的元素赋予类型
        ...
    }
    ...
}
```

当然，你也可以用栈来模拟参数的传递。复杂的分析情况下，你可以使用多个栈一起工作。

#### 为树节点添加字段

如果我们不在乎将用户编程语言与语法绑定，我们可以在规则文件中使用`[]` 为特定规则对应的树节点添加一个成员变量，例如

```
// 使用规则属性locals
tag locals [String name]: ID;
...
// 仅仅使用[]
defList[boolean inStruct]: def[$inStruct]*;
compSt: LC defList[false] stmtList RC;
structSpecifier: STRUCT optTag LC defList[true] RC | STRUCT tag;
```

使用`locals` 只会让`[]` 中标记的字段成为对应树节点的成员变量&#x20;

只使用`[]` 则不仅让`[]` 中标记的字段成为对应树节点的成员变量，还限制其必须在树节点构造函数中被初始化，构造函数的参数需要由其父节点的构造方法传入。

以上的代码中，由于`defList` 会出现在结构体中也可能出现在普通变量定义时，所以传入一个`boolean isStruct` 来标记其位置

你可以在Antlr自动生成的`CmmParser` 类中看到上述描述的具体实现

具体规则属性的使用可以参考[Rule Attribute Definitions](https://github.com/antlr/antlr4/blob/master/doc/parser-rules.md#rule-attribute-definitions)。

#### 使用Map

当然，我们可以使用一个全局的Map来将任意的值和树节点关联起来，父节点就可以通过子节点来访问到子节点关联的值，Antlr为我们提供了一个简单的帮助类`ParseTreeProperty`&#x20;

```java
public class CmmSemanticListener extends CmmParserBaseListener{
    ParseTreeProperty<XXX> values = new ParseTreeProperty<>();
    ...
    // specifier: structSpecifier| TYPE;
    @Override public void exitSpecifier(CmmParser.SpecifierContext ctx) {
        ...
        // 得到这个specifier对应的具体类型对象后，与当前specifier绑定
        values.put(ctx, type);
        ...
    }
    ...
    // paramDec: specifier varDec;
    @Override public void exitParamDec(CmmParser.ParamDecContext ctx) {
        ...
        // 得到子树specifier绑定的类型对象
        XXX type = values.get(ctx.specifier());
        // 给vardec中得到的ID或者数组中的元素赋予类型
        ...
    }
    ...
}
```

如果你想使用自己定义的Map类，请确保该类从`IndentityHashMap` 中派生，并确保判断树节点对象是否相同的条件正确。

#### 提示

1. 以上方法除了第一种仅使用于visitor模式外，其他的均适用于visitor与listener模式
2. 你可以在你的代码中使用以上一个或者多个方法
3. 一些规则备选分支数目较多，可以合理使用[Alternative Labels](https://github.com/antlr/antlr4/blob/master/doc/parser-rules.md#alternative-labels)

## 实验说明

报错时仅需要报出最本质的错误，并且尽可能的进行合理恢复，请留意报错的层级。

本次实验代码量偏大，请大家保持良好的面向对象编码风格，系统地设计类的职责以及各个类之间的调用关系。

若发现本文档有误或者考虑不周全，你可以与助教联系。
