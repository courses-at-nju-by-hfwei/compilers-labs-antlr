---
description: Antlr中间代码生成实验指导
---

# L4-Intercode-Reference

本次实验的实现逻辑与你使用的编程语言和工具基本无关，本讲义只是对Antlr部分与Java部分进行额外补充，详细内容请参考实验讲义Project\_3.pdf

## 实验介绍

当输入的Cmm代码经过之前的上下文无关分析和上下文有关分析并没有错误之后，我们就可以粗略地认为当前的代码是可以运行的了。编译器的下一步通常就是将用户输入的源代码转化为一种中间代码的形式，这样可以将编译器的前后端解耦并有利于编译器的模块化。

本次实验的主要内容就是进行中间代码的转换，你需要遍历之前实验中生成的语法树，借助语义分析中生成的符号表，在遇到不同的语法树节点的时候按照翻译范式生成不同的中间代码语句。

## 实验步骤

### 初始化符号表

我们假定在Cmm语言中有两个预定义函数`read`和`write` ，在Cmm代码中可以直接调用这两个函数以满足Cmm程序运行时与控制台的交互。

为了通过之前的语义分析，我们需要在语义分析之前，初始化符号表的时候将这两个函数添加到符号表中，下面对两个函数进行简要介绍，以便大家构造满足条件的类型

* 函数`read` 没有参数，每次会从控制台读取一个整数，分隔符为空格或者回车，返回值为从控制台读取的整数常量。
* 函数`write` 有一个整数类型的参数，该函数会将传入的整数打印到控制台中，返回值固定为0

在后续翻译Cmm代码的时候，遇到这两个函数也会翻译为特定的`READ xxx`和`WRITE xxx` 中间代码，中间代码运行程序在执行到这两行代码的时候，就会负责与控制台进行交互。

### 设计中间代码数据结构

> _有关中间代码的介绍可以参考许畅老师的实验讲义Project\_3.pdf的4.2.1_

在生成中间代码的时候，我们一般不会生成一条就打印一条，这样就不能在后续对于整体的中间代码进行调整和优化。一般的做法是将其存储在某种数据结构里，遍历完语法树之后再统一进行打印。

中间代码的表示有树形和线形，由于最后输出的中间代码就是线形的结构，所以为了方便，我们推荐大家统一使用线形的中间代码表示，线形表示又分为链表形式和数组形式，前者只比后者多了一个成员变量，用来指向下一行中间代码实例，下面给出链表形式的示例代码

```java
public abstract class InterCode {
  	// 枚举类，可以参考Project_3.pdf的表1
    CodeKind codeKind;
  	// 指向下一条中间代码
    InterCode next;
  	...
    public static class MonoOpInterCode extends InterCode {

        Operand operand;
      	...
    }
    public static class BinOpInterCode extends InterCode {

        Operand operand1;
        Operand operand2;
        Operand result;
      	...
    }
  	public static class AssignInterCode extends InterCode {

        Operand leftOperand;
        Operand rightOperand;
      	...
    }
    public static class ConditionJumpInterCode extends InterCode {

        Operand operand1;
        String relop;
        Operand operand2;
        Operand label;
				...
    }
 		...
    public static class MemDecInterCode extends InterCode {

        Operand operand;
        int size;
      	...
    }
  	...
}

public class InterCodeList {
  
  InterCode interCodeHead;
  InterCode interCodeTail;
  ...
}
```

根据不同中间代码的操作数个数以及需要的其他数据，可以将中间代码分为以上五种结构，操作数`Operand`的示例代码如下：

```java
public class Operand {

    OperandKind operandKind;
    String value;
  	...
}
public enum OperandKind {
    // 变量
    VARIABLE,
    // 常量
    CONSTANT,
    // 地址
    ADDRESS,
    // 跳转标签
    LABEL,
    // 
    FUNCTION
}
```

你下一步需要考虑的就是中间代码的合并函数，不同中间代码的打印函数，操作数的打印函数等等。

以上只是代码示例，请根据讲义仔细理解其中的内容，后续你也可以根据你的代码逻辑修改任何内容。

### 遍历语法树并生成中间代码

> _本次实验对输入的Cmm语言增加了更多限制，请参考许畅老师的实验讲义Project\_3.pdf的4.1.1中的7个假设_

在定义完相关的数据结构之后，你就需要再次对语法树进行遍历，并且对于不同的语法树节点生成不同的中间代码语句，注意，一个语法树节点可能会生成多行中间代码语句

### 打印中间代码

> _中间代码的语法请参考许畅老师的实验讲义Project\_3.pdf的表1_

## 实验说明

\
\
