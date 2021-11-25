---
description: Antlr语法分析实验指导
---

# L2-Parser-Reference

## 实验介绍

本次实验的内容需要在L1的基础上，对输入的c--代码进行语法分析，Antlr会在Parser中根据你定义的语法规则构建树，在构建树的过程中如果遇到错误则会进行错误识别，错误上报和错误恢复。

实验要求你能够修改错误上报的信息并打印，并且在没有遇到错误的时候能够通过listener模式或者visitor模式遍历Antlr自动构建的语法树，并且按要求打印出树的信息。

## 实验步骤

### 编写语法规则文件

#### 语法规则文件

> 语法规则的书写可参考[Antlr-Parser-Rules](l2-parser-reference.md#shi-yan-jie-shao) 以及_The Definitive ANTLR 4 Reference (2nd Edition) Chapter 15.3 _

在创建完CmmParser.g4之后，为了表明其只具备语法规则，并且引入CmmLexer.g4中定义的各种词法单元，需要在文件顶部声明如下

```
parser grammar CmmParser

options {
  tokenVocab=CmmLexer;
}
```

IDE中如果报错没有找到tokens file，请先进行`antlr4 CmmLexer.g4` 生成需要的词法文件

之后请根据附录的语法书写语法规则。

#### 语法规则

⚠️ 语法规则的正确与否决定了你这次实验的绝大部分得分。

与Bison不同的是，Antlr4会生成的语法分析器基于升级版的LL(\*)，也叫做ALL(\*)，解决了文法左递归的问题，并且支持了扩展的BNF语法。这也允许对附录给出的C--语言文法进行一定的升级，即各种`xxxList` 的规则不选择递归，而选择使用`*` ，下面将给出两个例子:

对于Program的规则，按照附录，在`CmmParser.g4` 文件中将会是如下内容

```
program: extDefList;
extDefList: extDef extDefList
    | 
    ;
extDef: specifier extDecList SEMI
  | specifier SEMI
  | specifier funDec compSt
  ;
```

由于Antlr4支持扩展BNF，需要将其改造成这样：

```
program: extDef* EOF;
extDef: specifier extDecList SEMI
  | specifier SEMI
  | specifier funDec compSt
  ;
```

注意，这里去掉了`extDefList` 并直接将`extDef*` 作为`program` 规则而不是修改`extDefList` 的规则，并且在program规则后面添加了EOF

这是为了parser以program规则为起点的时候，尽可能的读取输入，直到EOF，其余地方请直接修改`xxxList` 的规则即可

对于ExtDecList的规则，按照附录，在`CmmParser.g4` 文件中将会是如下内容

```
extDecList: varDec
  | varDec COMMA extDecList
  ;
```

需要将其改造成这样

```
extDecList: varDec (COMMA varDec)*;
```

除了上方的例子，附录的文法中还有几处需要修改，请自行完成。

### 打印语法树

在编写完CmmParser.g4之后，你已经定义出了C--语言，接下来关键就在于用语法分析器创建一颗语法树，然后使用特定程序代码遍历这颗语法树，在遍历树的过程中触发一定的事件。

你可以手动编写代码来遍历这颗树，也可以使用Antlr提供的树遍历工具或者树访问工具来遍历这颗树。

下面我们将介绍visitor和listener两种机制来获取树的信息。

⚠️ 在打印树信息的时候，你将会接触到许多类和接口，例如`ParserRuleContext` `TerminalNode` `Token` `RuleNode` `RuleContext` `ParseTree` 等等，请务必理清他们的继承、实现、依赖关系，以免混淆

#### Listener

> 具体例子请参考_The Definitive ANTLR 4 Reference (2nd Edition) Chapter4.3, 7.2_

使用listener会减少人为访问犯错的概率，但是灵活性不足，无法针对特定情况来自定义遍历的起点以及顺序。

listener机制一般会使用Antlr自带的`ParseTreeWalker` 在建立好语法树过后来访问它的所有的节点，在每次访问和离开节点的时候会分别触发对应规则子树的`enter`和`exit` 方法。

具体使用如下

```java
// Main.java
public static void main(String[] args){
    ...
    ...
    Parser parser = new CmmParser(tokens);
    ParseTree tree = parser.program(); 
    ParseTreeWalker walker = new ParseTreeWalker();
    // 类名请更换为你自己实现的Listener
    XXXListener listener = new XXXListener();
    walker.walk(listener, tree);
    ...
}
```

`walk` 方法接受一个`ParseTreeListener` 类型的参数，你需要实现一个listener类，满足它或者它的父类实现了`ParseTreeListener` 接口，并且按需要重写其中的方法，然后将自定义listener的对象传入`ParseTreeWalker` 的`walk` 方法中，关注`walk` 方法：

```java
// ParseTreeWalker.java
public void walk(ParseTreeListener listener, ParseTree t) {
    ......
    if ( t instanceof TerminalNode) {
        listener.visitTerminal((TerminalNode)t);  // 访问词法单元
        return;
    }
    RuleNode r = (RuleNode)t;
    enterRule(listener, r);        //  类似于先序遍历
    int n = r.getChildCount();
    for (int i = 0; i<n; i++) {
        walk(listener, r.getChild(i));
    }
    exitRule(listener, r);         //  类似于后序遍历
}
```

此方法实现了对语法树的深度优先遍历，对叶节点(TerminalNode)进行访问，以及触发进入和退出非叶节点(RuleNode)的规则。

进一步查看代码，你就能找到本次实验需要override的内容。

#### Visitor

> 具体例子请参考_The Definitive ANTLR 4 Reference (2nd Edition) Chapter4.2, 7.3_

visitor模式下，你并不需要使用一个额外的walker来遍历整棵树，而是使用一个实现了`ParseTreeVisitor` 接口的类来完成所有访问的操作，这意味着对所有节点的访问必须要显式调用，一旦忘记对某个节点进行访问，就会导致整棵树的遍历失败。但visitor提供了listener所不具备的灵活性，你可以自定义节点访问的顺序以及节点访问与否，就需要更加关注访问的细节。

当在命令行中使用- visitor参数的时候，Antlr就会生成额外的`CmmParserVisitor` 接口以及`CmmParserBaseVisitor` 类

前者继承了`ParseTreeVisitor` 接口，并新加了对C--语言规则的访问方法，后者实现了前者，并给C--语言规则的访问方法赋予了默认实现，并且继承了抽象类`AbstractParseTreeVisitor` ，这个类中有对`ParseTreeVisitor` 接口的默认实现

访问器的起点一般都是通过`T visit(ParseTree tree)` 方法开始的，也就是传入一棵实现了`ParseTree` 接口的树对象，然后开始对整颗树进行访问，他的默认实现在`AbstractParseTreeVisitor` 中

```java
// AbstractParseTreeVisitor.java
/**
 * {@inheritDoc}
 *
 * <p>The default implementation calls {@link ParseTree#accept} on the
 * specified tree.</p>
 */
@Override
public T visit(ParseTree tree) {
	return tree.accept(this);
}

```

这里传入的树可以是`TerminalNode` 的实现`TerminalNodeImpl` (Antlr定义的，是所有叶节点的实现），也可以是`RuleNode` 的实现。

调用`accept`方法，在`accept` 方法中控制`visitor` 的行为，**默认情况下**，对于非叶节点，就会调用visitor的`visitChildren()` 对于叶节点，就会调用visitor的`visitTerminal()` 。

总之，visitor的让树的遍历过程可以完全依赖于用户，你可以定义出不同的遍历方法，使用大体如下

```java
// Main.java
    public static void main(String[] args) throws Exception {
        ...
        ...
        ParseTree tree = parser.program();
        XXXVisitor visitor = new XXXVisitor(); // 替换成你自己实现的visitor类
        visitor.visit(tree);
        ...
        ...
    }


```

清楚调用过程后，你应该就能知道本次实验需要override的内容。

### 错误的识别和恢复

一旦拥有了正确的语法，就必须处理不合语法的语句。人们不希望一个语法分析器对于非法输入的响应仅仅停留在遇到一个语法错误就会退出，而是能尽可能的识别错误，恢复错误，然后继续检查之后的错误。

#### 错误的打印

> 具体请参考_The Definitive ANTLR 4 Reference (2nd Edition) Chapter 9.2_

在本次实验中，大家并不需要关心如何编写识别错误的代码，如何定位错误以及如何恢复错误。Antlr强大的错误报告功能和复杂的错误恢复机制已经帮你完成了百分之九十的工作，你可以尝试着往你的程序中输入一下错误代码：

```c
int main(){
  int i
  int j;
  int k
}
```

如果你的语法规则文件编写无误，你应该会收到一下报错信息:

```
line 3:2 missing ';' at 'int'
line 5:0 missing ';' at '}'
```

Antlr的错误报告功能在发现错误过后打印出错误信息，并且借助于自带的错误恢复机制，继续识别出了接下来的错误。所以大家接下来需要做的就只是修改其默认的错误信息，变为本次实验需要的格式。

观察Antlr构建语法树的全过程，可以看到在Antlr自动生成的CmmParser中，Antlr会为每个规则生成一个方法，用于构造该规则下的语法树，并且返回其构造的树的根节点，例如`public final ProgramContext program() throws RecognitionException` ，这些方法都通过了一个`try...catch...finally` 结构来实现内部逻辑。

```java
public final ProgramContext program() throws RecognitionException {
	...
	try {
		...
	}
	catch (RecognitionException re) {
		_localctx.exception = re;
		_errHandler.reportError(this, re); 
		_errHandler.recover(this, re);
	}
	finally {
		...
	}
	...
}
```

在`try` 代码块中实现了备选规则的选择与匹配以及对应子树的构造，`catch` 中的代码主要负责错误报告和错误恢复。

`_errorHandler` 是一个`DefaultErrorStrategy` 类型的对象，实现了`ANTLRErrorStrategy` 接口，对应的`reportError()` 函数代码中，根据`CmmParser` 中抛出的错误生成了不同的错误信息，在`DefaultErrorStrategy`的其他函数代码中，也会根据错误恢复的情况产生其他错误信息，这里不过多阐述。

在`DefaultErrorStrategy`的这些用于于生成错误信息的函数的最终，都会调用传入的`CmmParser` 对象的`notifyErrorListeners` 方法，大体内容如下

```java
String msg = ...
recognizer.notifyErrorListeners(...);
```

所以，回到`notifyErrorListeners` 方法，可以看到它通过`getErrorListenerDispatch();` 获取到了一个错误监听器的代理对象`ProxyErrorListener` （负责所有listeners的行为）

这个代理对象中就有`Parser` 对象中的`ANTLRErrorListener` 对象列表`_listeners` （由`Recognizer` 抽象类继承而来）

这个列表中有一个默认的`ANTLRErrorListener` 接口实现对象：`ConsoleErrorListener` ，在这个类中，你应该就能看见上述报错信息的`line 3:2` 以及`line 5:0` 是如何生成的了。

所以，如果你想要将报错信息修改为实验要求的样子，你就可以模仿`ConsoleErrorListener`的写法，自定义一个错误监听器，并且重写`syntaxError()` 方法，然后添加到你的`CmmParser` 对象的`ANTLRErrorListener` 对象集合`_listeners` 中。

其实在`CmmLexer` 中也有继承而来的`_listeners` 列表，L1实验中，也可以通过自定义错误监听器的方式来完成词法错误打印。

#### 勘误备选分支

> 具体请参考_The Definitive ANTLR 4 Reference (2nd Edition) Chapter 9.4_

在编写语法分析器的时候，我们会对一些非常常见的语法错误进行特殊处理，这是非常值得的，通过接受这些常见的错误的方式，通过手动报错，减少了Antlr自动报错以及自动恢复的开销，并且能够得到更精确的报错信息。

本次实验中，我们需要大家再实现**一个错误备选分支**，即发生该类错误的时候不会去触发Antlr的report-recover机制，而是进入自定义的错误备选分支，默认该错误是语法允许的，然后再手动报告该错误信息。

例如对于如下代码：

```java
int main(){
  int a[1][n][3][1.3] = 3; // 声明数组的下标只能使用INT类型的常量
  int i = 0;
  int j = 1;
}
```

将其输入你的程序，语法正确的情况下，Antlr默认会报如下错误：

```
line 2:11 no viable alternative at input 'a[1][n'
line 3:2 missing '}' at 'int'
```

这样的报错信息无疑是不清楚并且冗余的，我们需要你的程序对于该输入能够只报一个错误，并且打印自定义的，更加细节的报错信息，例如

```
Error type B at Line 2: array size must be an integer constant, not n
```

当然，你也可以再细节化你的报错信息。

⚠️ 使用勘误备选分支会让Antlr程序认为该错误是正确的文法，构建一颗完整的语法树。如果整个代码只有这一种错误，请不要让你的程序打印树的结构，而是打印报错信息。

⚠️ 我们确保用例中的`[]` 内出现且仅出现一个 `ID`，`INT`，`FLOAT`三种类型之一的Token。

## 实验说明

本实验文档意在为大家在本次实验中提供充分的发挥空间，所以并没有具体地说明实现方式，看起来不会如L1那么易懂，请大家充分实践课堂以及书本上的例子，阅读源码，研究继承实现依赖关系。

鼓励大家在实验报告中记录你的实现方法以及钻研过程。

若发现本文档有误或者考虑不周全，你可以与助教联系。



