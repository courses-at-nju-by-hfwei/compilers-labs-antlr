# Develop-in-IDEA-Antlr

本文档的目的在于在IDEA上帮助大家创建一个与本次实验Lab相适配的项目，以便在实验开发过程中使用IDEA提供的丰富的插件和辅助功能，更快捷的完成实验。

以下提供了三种部署方式，推荐使用Maven/Gradle方式部署，这样可以在每个package下放置不同的Antlr项目，除了本次Lab的每次实验作业`L1/L2/L3...` ，也可以创建课堂上的示例项目等等。

⚠️ 如果你已经有自己的coding流程，能够测试，能够提交，请忽略此文档。

⚠️ 本文档默认你在MacOS或者Windows下使用IDEA，实验的Lab在Linux环境目录里。

## Directly Import

### 环境配置

用IDEA直接open实验的`Lab`文件夹 (如果你是使用Linux虚拟机获取的实验Lab，请将Lab文件夹放到与宿主机共享的文件夹中)

在`src`下创建.g4文件，然后各进行少量的编码

![](.gitbook/assets/Snipaste\_2021-11-15\_20-12-44.png)

配置.g4文件的output路径。根据实验的要求，我们直接配置到`src`路径下，右键`.g4` 文件，点击Configure ANTLR，修改Output directory到项目的src目录。并勾选`generate parse tree visitor` (方便之后的实验）

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-16-18 (1).png>)

之后右键.g4文件，点击Generate ANTLR Recognizer，Antlr自动生成的文件就会放在src目录下

⚠️如果目录下有多个.g4文件，请对每个.g4文件执行上述操作

接下来的工作是将 [antlr4-4.9.2-complete.jar](https://repo1.maven.org/maven2/org/antlr/antlr4/4.9.2/)导入到项目中，菜单栏-> File-> Project Structure，添加jar包

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-20-15 (1).png>)

执行如上操作过后，可以注意到你的java代码里将不会有红色报错信息，通过`Main.java` 的入口函数`main` 执行整个项目后，我们注意到idea额外生成了一个out文件夹，请将其加入到`.gitignore` 文件中

添加的大致内容如下

```
/out/*
Lab.iml
.idea
```

如果后期的实验又有额外的二进制文件产生，请以同样的方法把它们添加到`.gitignore`中

### 项目目录

由于是IDEA自动打开的本次实验项目，项目目录并不需要配置

### 提交代码

在你编写完每次实验需要的内容，IDEA调试正确后，请在项目要求的Linux环境下，在terminal中打开本Lab文件夹，然后进行`make` / `make submit` / `make run FILEATH=xxxx.cmm` 等操作

## Maven

### 环境配置

在IDEA里，File->New Project-> Maven-> Project SDK可以选择1.8+的版本->next，然后输入项目的路径和名称

![](<.gitbook/assets/image (1) (1).png>)

在`pom.xml` 文件中添加如下内容，然后点击右上角的图标(Load Maven Changes)

```markup
  <dependencies>
    <dependency>
      <groupId>org.antlr</groupId>
      <artifactId>antlr4-runtime</artifactId>
      <version>4.9.2</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.antlr</groupId>
        <artifactId>antlr4-maven-plugin</artifactId>
        <version>4.9.2</version>
        <executions>
          <execution>
            <id>antlr</id>
            <goals>
              <goal>antlr4</goal>
            </goals>
            <phase>none</phase>
          </execution>
        </executions>
        <configuration>
          <outputDirectory>src/test/java</outputDirectory>
          <listener>true</listener>
          <treatWarningsAsErrors>true</treatWarningsAsErrors>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>8</source>
          <target>8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

在Plugins中搜索ANTLR，安装对应插件

![](.gitbook/assets/image.png)

### 项目目录

在`src/main/java`目录下，可以创建每次实验的package

例如L1实验，就可以创建一个`L1` package，在目录下增加本次实验需要的`Main.java` 以及`CmmLexer.g4` 文件。(package的内容与每次实验Lab的src应该具有相同的文件内容，L1实验就至少需要这两个文件）

右键`CmmLexer.g4`，选择Configure ANTLR，填入output directory(选择至java目录即可)和pacakge的内容(L1)，勾选`generate parse tree visitor` (方便之后的实验）

![](<.gitbook/assets/image (2).png>)

编写完`CmmLexer.g4`后，右键`CmmLexer.g4` ，选择Generate ANTLR Recognizer。即可在本package下看到自动生成的文件

之后就可以根据生成的Java类编写实验代码

⚠️如果目录下有多个.g4文件，请对每个.g4文件执行上述操作。

### 提交代码

将你每次实验对应的package下自定义的java代码，以及`Main.java` 还有`.g4` 文件**复制**到实验的Lab/src/下，并删除所有Java代码顶部与你的package有关的任何import和声明。

⚠️ 如果你的自定义代码中使用到了一个类的内部类，例如`CmmParser` 的内部类`xxxContext` ，请在代码中使用`CmmParser.xxxContext` 而不是`xxxContext` 来引用这个类，以免编译时报错找不到该类。

然后在Lab目录下输入`make` /`make test`/`make run FILEPATH=xxx.cmm` 调试，如果没有问题，请选择`make submit` 提交。

## Gradle

与Maven近似，不过多叙述。`build.gradle` 文件可参考：

```
plugins {
    id 'java'
    id 'antlr'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    antlr 'org.antlr:antlr4:4.9.2'
    implementation group: 'org.testng', name: 'testng', version: '6.14.3'
}

// ['-package', 'list']
// "-long-messages"
// "-atn"
generateGrammarSource {
    arguments += ["-Xlog", "-visitor"]
}

test {
    useTestNG() {
        useDefaultListeners = true
    }
}
```
