---
description: Antlr词法分析实验指导
---

# L1-Lexer-Reference

本讲义在于对实验讲义Project\_1.pdf词法分析部分的额外补充，以便更快使用Antlr进行开发（并不意味着你们不需要阅读实验讲义部分）。

## 实验介绍

​ 本次实验一任务是编写一个程序对使用C--语言书写的源代码进行词法分析，C--语言的语法已经在附录给出。实验要求使用Antlr工具进行词法分析工作，并且能够借助其生成的java文件来完成特定语句的打印。

​ 在Antlr实验中，我们通常会将lexer和parser的结合在一起，并且通过Listener模式或者Visitor模式对于生成的Parse Tree进行用户自定义的操作。

​ 本次实验，我们将只实现Lexer部分，本实验说明将会提供一定指引来完成对于lexer的独立访问和自定义操作。

## 实验步骤

### Antlr规则文件

​ 在创建完CmmLexer的操作之后，为了表明其只具备词法规则，我们需要在文件顶部声明如下

```
lexer grammar CmmLexer;
```

​ 随后，你就可以根据附录的词法部分以及实验相关要求在后续书写C--语言的词法规则（推荐多使用`fragment` 字段）。

​ Antlr的语法规则文件`*.g4`格式可参考[Antlr-Lexer-Rules](https://github.com/antlr/antlr4/blob/master/doc/lexer-rules.md) 以及_The Definitive ANTLR 4 Reference (2nd Edition)_ 的5.5以及15.2章节。

### Main.java

​ 在实现完`CmmLexer.g4`文件后，使用如下命令生成Antlr自动构造的工具类

```shell
$ antlr4 -listener -visitor CmmLexer.g4 # 也可以不添加任何选项，本次实验中不会有任何影响
```

​ 你将在src目录下看到生成的`CmmLexer.java` 等文件，之后便可以在`Main.java`中依据该类构造你自己的代码。

```java
import org.antlr.v4.runtime.*;
import java.io.*;
import java.util.*;


public class Main
{
    public static void main(String[] args){
      	...
        ...
        CmmLexer lexer = new CmmLexer(input);
      	...
        ...
    }
}
```

​ 在Main函数中，你需要args中读取文件的路径，并且读取文件的内容（你在L0中已经实现的内容），最终传入到`CmmLexer`中构造对象。

​ 所以我们需要做的就是让lexer对象从头到尾读取input，并且分析出每个词法单元，如果有错误就打印错误，否则打印所有的词法单元。

### 错误信息打印

#### 实现方式1

​  我们通过观察源码，可以看到在`org.antlr.v4.runtime.Lexer`类中有这样的方法，antlr自动生成的`CmmLexer`继承自该类

```java
public List<? extends Token> getAllTokens() {...}
```

​ 这个方法通过while循环，不断的调用`nextToken()` ，以识别下一个语法单元，并且最终返回TokenList，本次实验需要完成任务之一就是在正确读取词法单元后，打印词法单元的信息，如果有不满足任何词法规则的单元，则报错。

​ 在`nextToken()`方法中，需要注意到的是以下代码

```java
public Token nextToken() {
    if (_input == null) {
        throw new IllegalStateException("nextToken requires a non-null input stream.");
    }
    ...
    try{
        ...
        while (true) {
            ...
            do {
                ...
                int ttype;
                try {
                    ttype = getInterpreter().match(_input, _mode);
                }
                catch (LexerNoViableAltException e) {
                    notifyListeners(e);		// report error
                    recover(e);
                    ttype = SKIP;
                }
                ...
            } while ( _type ==MORE );
            ...
        }
    }
    finally {
        ...
    }
}
```

​ 代码大致意思如下：如果当前识别出的`_input`(CharStream类型)并不能满足lexer中定义的任何词法单元时，将会报错，然后返回没有正确分类的Token，否则返回相应Token。

​ 在报错后，将会调用Lexer本身的`notifiListeners(e)` 方法，传给默认的监听器，默认情况下，控制台将会打印类似于如下的信息

```
line 1:0 token recognition error at: '*'
line 1:1 token recognition error at: '*'
line 1:2 token recognition error at: '*'
```

​ 为了能够自定义打印的信息，我们需要对`notifiListeners(e)` 方法进行Override，一个合适的时机就是在生成CmmLexer对象的时候。所以上面得到Lexer对象的代码就会变为：

```java
import org.antlr.v4.runtime.*;
import java.io.*;
import java.util.*;


public class Main
{
    public static void main(String[] args){
      	...
        ...
        CmmLexer lexer = new CmmLexer(input) {
            @Override
            public void notifyListeners(LexerNoViableAltException e) {
              // TODO
            }
        };
      	List<? extends Token> tokenList = lexer.getAllTokens();
      	// TODO
      	...
        ...
    }
}        
```

​  在TODO中，就可以调用`_input`的方法获取错误的text以及错误发生的行号，最后生成错误语句，例如获取不能识别的text可以如下：

```java
String text = _input.getText(Interval.of(_tokenStartCharIndex, _input.index()));
```

&#x20; 最后，按照要求生成对应的报错语句，如果没有报错语句的生成就逐行打印词法单元的信息。

​  至此，我们就实现了词法单元Token的识别与错误词法单元报错。

#### 实现方式2

&#x20; ​除了从CmmLexer中获取tokenList，也可以将CmmLexer传入词法单元流中例如`CommonTokenStream` 当中

```java
public class Main
{
    public static void main(String[] args) {
        ...
        ...
        CmmLexer lexer = new CmmLexer(input){
            @Override
            public void notifyListeners(LexerNoViableAltException e) {
              // TODO
            }
        };
        CommonTokenStream tokenStream = new CommonTokenStream(lexer);
        // TODO
        ...
    }
}
```

​  然后通过`CommonTokenStream` 的成员函数读取Token的内容，如`LT(int)` 以及`LB(int)` 来逐个从输入文件内容中读取词法单元，直到EOF。也可以先调用`fill()` 获得所有的词法单元，填充进成员变量`List<Token> tokens` 当中，再一次性读取。



⚠️ 无论什么方式，在读取完所有词法单元的时候，读取文件的“指针”已经指向了文件内容的最后，想要再度从Lexer中从头读取，请使用Lexer中已经定义好的复位函数（或者新建一个Lexer对象）。

⚠️ 由于Antlr默认的词法错误识别规则，若不满足的字符序列在行末，从错误Token中读取到的text中会带上该行的换行符号 ，所以可以使用`text.trim()`来进行调整。如果没有`trim()`函数，或许会有以下现象(以下词法中没有定义id）：

```
&
flo
flaot

Error type A at Line 1: undefined symbols &\n .
Error type A at Line 2: undefined symbols flo\n .
Error type A at Line 3: undefined symbols fla .
```



### 词法单元打印

如果没有检测出词法错误，你需要按序打印每一个`Token`的详细信息。观察antlr自动生成的`CmmLexer`类的源码，可以发现其已经为我们构建好了一个静态的`Vocabulary`对象。因此可以使用以下方式获得`Token`的符号名称。

```
CmmLexer.VOCABULARY.getSymbolicName(token.getType());
```



你可以在提交的实验报告中，提出你所采用的，与上述实现方式不同的本次实验的其他实现方法。

若发现本文档有误或者考虑不周全，你可以与助教联系。

## 实验说明

* 本次实验在环境部署正确的情况下，不使用任何IDE也能完成实验。
* 若使用IDE例如idea等（方便查看antlr源码）编写代码，请新建项目，并在调试成功后，将实现的源代码文件放入本实验的src目录下，并且删除顶部可能会存在的package声明，以及无用的import，使用make调试成功后，进行`make submit`
* 请认真阅读实验说明，实验一讲义，以及附录。

​
