---
description: Antlr中间代码生成实验指导
---

# L4-Intercode-Reference

本次实验的实现逻辑与你使用的编程语言和工具基本无关，本讲义只是对Java代码部分进行额外补充，详细内容请参考实验讲义Project\_3.pdf

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
    // 函数
    FUNCTION
}
```

你下一步需要考虑的就是中间代码的合并函数，不同中间代码的打印函数，操作数的打印函数等等。

以上只是代码示例，请根据讲义仔细理解其中的内容，后续你也可以根据你的代码逻辑修改任何内容。

### 遍历语法树并生成中间代码

> _本次实验对输入的Cmm语言增加了更多限制，请参考许畅老师的实验讲义Project\_3.pdf的4.1.1中的7个假设_

在定义完相关的数据结构之后，你就需要对语法树进行遍历，对于不同的语法树节点生成不同的中间代码语句段。

你可以将生成中间代码的逻辑直接放到之前用于语义分析的`visitor` 或者`listener` 代码中，这样遍历一遍语法树就可以完成语义分析和中间代码生成工作，效率较高

但是访问树节点的方法将会变得非常复杂，并且需要处理多种类型变量的传递和返回，而且在生成中间代码的时候也不需要访问语义分析那么多的树节点。

所以我们还是推荐新建一个`visitor`和`listener` ，并以`InterCodeList` 类型以及`Operand` 类型的对象作为树节点之间需要传递的值，如何传递在L3中已经有了很详细的示范，这里就不再赘述，请灵活选用多种方法来实现你的翻译逻辑。

如何翻译特定的树节点，在_Project\_3.pdf_的_4.2.5, 4.2.6, 4.2.7, 4.2.8_中有很明确的说明（其中的translate_\__xxx就可以理解为Antlr中的`visitXXX` 以及`enterXXX` 和`exitXXX` ），请仔细阅读

请务必在理解之后再尝试着编写Java代码，你也需要适当修改一些语法单元的翻译模式，因为我们某些节点使用的语法规则并不与文中所使用的语法规则相同（例如`args` ），修改传参，将`sym_table` 作为全局变量，避免在树节点中的传递。

注意到在翻译`exp`的时候需要额外传入一个操作数`place` 用于存放`exp` 计算出的表达式，请考虑`place` 的生成时机以及类型

若需要翻译`exp: exp1 LB exp2 RB` ，即访问数组，翻译逻辑可以为

1. 访问`exp1` 并传入创建的`baseAddr` 操作数，生成获取`baseAddr` 的中间代码
2. 访问`exp2` 并传入创建的`index` 操作数，生成获取`index` 的中间代码
3. 通过某个函数得到`exp1` 对应的`Array` 的元素大小`element_size`&#x20;
4. 生成获取偏移量中间代码 `[offset = index * elementSize]`&#x20;
5. 获取地址并直接赋给`place` ，即中间代码 `[place := baseAddr + offset]` ，并将`place` 的类型设置为`ADDRESS` ；或者获取地址并取得值，再赋给`place` ，即中间代码`[realAddr := baseAddr + offset] + [place := *readAddr]` ，选择哪种取决于你整体的代码逻辑。

结构体成员的访问方式也可以参考上述的翻译逻辑。



再具体到实际的java代码编写，下面给出翻译`fundec` 节点的代码示范（使用`visitor` 模式），仅仅用作逻辑参考，代码可用性无法保证

```java
public class CmmInterCodeGenerator extends CmmParserBaseVisitor<InterCodeList> {
...
    @Override
    public InterCodeList visitFunDec(CmmParser.FunDecContext ctx) {
        // 获取函数名
        String functionName = ctx.ID().getText();
        // 新建一个存储该函数名的Operand
        Operand functionOp = new Operand(OperandKind.FUNCTION, functionName);
        // funcDefineCode打印的中间代码为: FUNCTION XXX:
        InterCode.MonoOpInterCode funcDefineCode = new InterCode.MonoOpInterCode(CodeKind.FUNCTION, functionOp);
        // 从符号表中获得当前函数名对应的函数Type，并获得这个函数定义的形参
        Field curParam = ((Function) table.getType(functionName)).getParamListHead();
        // 遍历形参
        while (curParam != null) {
            // 生成函数参数声明的中间代码: PARAM xxx
            Operand paramOp = new Operand(OperandKind.VARIABLE, curParam.getName());
            InterCode paramCode = new InterCode.MonoOpCode(CodeKind.PARAM, paramOp);
            // 添加到以funcDefineCode为头节点的链中
            funcDefineCode.addInterCode(paramCode);
            // 获取下一个形参
            curParam = curParam.getNext();
        }
        return new InterCodeList(funcDefineCode);
    }
...
}
```

接下来的任务就是根据前面介绍的翻译模式完成一系列的语法树节点翻译工作

除了我们已经给出示范的，你还需要考虑包括数组访问、结构体访问、数组与结构体定义、变量初始化、 语法单元CompSt、语法单元StmtList在内的其他翻译模式，其中一些的翻译工作非常简单，就是将子节点生成的中间代码合并，例如`program extDef compSt stmtList` 等等。

### 打印中间代码

> _中间代码的语法请参考许畅老师的实验讲义Project\_3.pdf的表1_

在翻译完整棵语法树后，理想的情况就是生成的所有中间代码都存放在了`InterCodeList` 对象中，只需要对其成员`InterCode` 进行遍历打印即可。

不同类型的`InterCode`对象有着不同的打印方式，你可以使用最简单的`if...else` 或者`swicth` 语句来进行打印工作，也可以尝试一些 java编程技巧来去掉大量的条件判断语句，鼓励大家在实验报告中描述你所采用的方法

## 实验说明

请认真阅读Project\_3.pdf，务必在完全理解翻译模式的基础上进行编码

若发现本文档有误或者考虑不周全，你可以与助教联系。

\
\
