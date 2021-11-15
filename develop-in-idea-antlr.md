# Develop-in-IDEA-Antlr

本文档的目的在于在IDEA上帮助大家创建一个与本次实验Lab相适配的项目，以便在实验开发过程中使用IDEA提供的丰富的插件和辅助功能，更快捷的完成实验。

⚠️ 如果你已经有自己的coding流程，能够测试，能够提交，请忽略此文档。

## Directly Import

#### 一、idea项目配置

1 . 用idea直接open项目的`Lab`文件夹 (如果你是在Windows平台下使用Ubuntu虚拟机，你可以将Lab文件夹放到与宿主机共享的文件夹中)；在`src`下创建.g4文件，然后各进行少量的编码。

![](.gitbook/assets/Snipaste\_2021-11-15\_20-12-44.png)

2\. 配置.g4文件的output路径。根据实验的要求，我们直接配置到`src`路径下

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-13-43 (3).png>)

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-16-18 (1).png>)

3\. 然后开始生成，CmmLexer.java文件就出现在src路径下。

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-16-50 (1).png>)

![](.gitbook/assets/Snipaste\_2021-11-15\_20-17-44.png)

4\. 接下来的工作是将 [antlr4-4.9.2-complete.jar](https://repo1.maven.org/maven2/org/antlr/antlr4/4.9.2/)  导入到idea中，这样idea就不会爆红了

![](.gitbook/assets/Snipaste\_2021-11-15\_20-18-18.png)

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-20-15 (1).png>)

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-22-26 (1).png>)

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-23-09 (1).png>)

#### 二、.gitignore 的书写

1 . 执行main后，我们注意到idea额外生成了一个out文件夹，里面有二进制.class文件以及.g4文件，但它还没被添加到.gitignore中......

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-25-47 (1).png>)

![](.gitbook/assets/Snipaste\_2021-11-15\_20-27-33.png)

2\. 如果你还不熟悉.gitignore的写法，可以用idea所提供的功能将多余的文件添加入.gitignore中

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-29-52 (1).png>)

![](.gitbook/assets/Snipaste\_2021-11-15\_20-30-13.png)

* 最终的.gitignore文件至少应该如上图所示，接下来就可以放心地使用`git add .` 等命令了

3\. 如果你还不放心，想验证自己的git配置是否成功。你可以到OJ平台上把submit.zip下载到本地的一个空文件夹内，然后执行`git reset HEAD --hard`  命令。正常情况下，你的提交不应该有过多额外的文件。

![](<.gitbook/assets/Snipaste\_2021-11-15\_20-58-43 (1).png>)

#### 三、Antlr4源码下载与阅读

1 . 源码通过[GitHub地址](https://github.com/antlr/antlr4/tree/4.9.2)获取，相应的Tag不要选错，然后直接下载其zip包到本地

![](<.gitbook/assets/image (1).png>)

2\. 回到idea，注意到当前我们所看到的antlr4代码都是反编译后的结果，因此局部变量名和注释是缺失的

![](<.gitbook/assets/Snipaste\_2021-11-15\_21-14-27 (1).png>)

3\. 选择"Choose Source"后，选择刚刚下载好的zip包，一路ok就行

![](<.gitbook/assets/Snipaste\_2021-11-15\_21-15-23 (1).png>)

![](.gitbook/assets/Snipaste\_2021-11-15\_21-15-50.png)

4\. 最终成功看到antlr4的注释等内容

![](<.gitbook/assets/Snipaste\_2021-11-15\_21-16-10 (1).png>)



⚠️ 此方法前期配置比较繁琐，请谨慎操作，**尤其是out文件夹**，请务必把它添加到**`.gitignore`**中

⚠️ 如果后期的实验又有额外的二进制文件产生，请以同样的方法把它们添加到`.gitignore`中

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

编写完`CmmLexer.g4`后，右键`CmmLexer.g4` ，选择Generate ANTLR Recognizer。即可在本package下看到自动生成的文件。

之后就可以根据生成的Java类编写实验代码。

### 提交代码

将你每次实验的package下自定义的java代码，以及`Main.java` 还有`.g4` 文件复制到实验的Lab/src/下，并删除所有Java代码顶部的package声明。

然后在Lab目录下输入`make` /`make test`/`make run FILEPATH=xxx.cmm` 调试，如果没有问题，请选择`make submit` 提交

## Gradle

TODO

