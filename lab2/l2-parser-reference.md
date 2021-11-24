---
description: TODO
---

# L2-Parser-Reference

## 实验介绍



## 实验步骤

### 编写语法规则文件

### 打印语法树

#### Listener

#### Visitor

### 错误的识别和恢复

一旦拥有了正确的语法，我们就必须处理不合语法的语句。我们并不希望一个语法分析器对于非法输入的响应仅仅停留在遇到一个语法错误就会退出，而是能尽可能的识别错误，恢复错误，然后继续检查之后的错误。

#### 错误的打印

在本次实验中，大家并不需要关心如何编写识别错误的代码，如何定位错误以及如何恢复错误。ANTLR强大的错误报告功能和复杂的错误恢复机制已经帮你完成了百分之九十的工作，你可以尝试着往你的程序中输入一下错误代码：

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

观察ANTLR构建语法树的全过程，可以看到在ANTLR自动生成的CmmParser中，ANTLR会为每个规则生成一个方法，用于构造该规则下的语法树，并且返回其构造的树的根节点，例如`public final ProgramContext program() throws RecognitionException` ，这些方法都通过了一个`try...catch...finally` 结构来实现内部逻辑。

```
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

在编写语法分析器的时候，我们会对一些非常常见的语法错误进行特殊处理，这是非常值得的，通过接受这些常见的错误的方式，通过手动报错，减少了ANTLR自动报错以及自动恢复的开销，并且能够得到更精确的报错信息。

本次实验中，我们需要大家再实现**一个错误备选分支**，即发生该类错误的时候不会去触发ANTLR的report-recover机制，而是进入我们自定义的错误备选分支，默认该错误是语法允许的，然后再手动报告该错误信息。

例如对于如下代码：

```java
int main(){
  int a[1][n][3][1.3] = 3; // 声明数组的下标只能使用INT类型的常量
  int i = 0;
  int j = 1;
}
```

将其输入你的程序，语法正确的情况下，ANTLR默认会报如下错误：

```
line 2:11 no viable alternative at input 'a[1][n'
line 3:2 missing '}' at 'int'
```

这样的报错信息无疑是不清楚并且冗余的，我们需要你的程序能够只报一个错误，并且打印自定义的，更加细节的报错信息，例如

```
Error type B at Line 2: array size must be an integer constant
```

当然，你也可以再细节化你的报错信息。

具体的实现请参考_The Definitive ANTLR 4 Reference (2nd Edition) Chapter 9.4 Error Alternatives_

⚠️ 使用勘误备选分支会让ANTLR程序认为该错误是正确的文法，构建一颗完整的语法树。如果整个代码只有这一种错误，请不要让你的程序打印树的结构，而是打印报错信息。

⚠️ 我们确保用例中的`[]` 内只会出现ID，INT，FLOAT三种类型的Token

## 实验说明
