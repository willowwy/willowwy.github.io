---
title: CodeQL安装与使用
date: 2023-07-27 16:36:52
tags: [ Codeql ]
categories: [ Tools ]
description: CodeQL安装与使用
cover: images/codeql.png
comments: false
---

# CodeQL使用

## CodeQL简介

Github为了解决其托管的海量项目的安全性问题，收购了CodeQL的创业公司，并宣布**开源CodeQL的规则**
部分，这样全世界的安全工程师就可以贡献高效的QL审计规则给Github，帮助它解决托管项目的安全问题。

对于安全工程师，也就多了一个**非商业的开源**代码自动化审计工具。

CodeQL支持非常多的语言，在官网有如下支持的语言和框架列表。
<img src="image-20221124173020619.png" alt="img" style="zoom:67%;" />

## CodeQL原理

代码转化成类似数据库的形式，并基于该database进行分析。

在 CodeQL 中，代码被视为数据。安全漏洞、Bug 和其他错误被建模为可针对从代码中提取的数据库执行的查询。

> CodeQL 的整体思路是把源代码转化成一个可查询的数据库，通过 Extractor 模块对源代码工程进行关键信息分析提取，构成一个关系型数据库。CodeQL
> 的数据库并没有使用现有的数据库技术，而是一套基于文件的自己的实现。
> 对于编译型语言，Extractor 会监控编译过程，编译器每处理一个源代码文件，它都会收集源代码的相关信息，如：语法信息（AST
> 抽象语法树）、语意信息（名称绑定、类型信息、运算操作等），控制流、数据流等，同时也会复制一份源代码文件。而对于解释性语言，Extractor
> 则直接分析源代码，得到类似的相关信息。
> 关键信息提取完成后，所有分析所需的数据都会导入一个文件夹，这个就是 CodeQL database, 其中包括了源代码文件、关系数据、语言相关的
> database schema（schema 定义了数据之间的相互关系）。

## CodeQL安装

CodeQL本身包含两部分 **解析引擎 + SDK** 。

解析引擎：**不开源**，解析我们编写的规则，但是可以直接在官网下载二进制文件直接使用。

SDK：**完全开源**，里面包含大部分现成的漏洞规则，我们也可以利用其编写自定义规则。

### 解析引擎安装

URL: https://github.com/github/codeql-cli-binaries/releases

下载已经编译好的codeql执行程序，解压之后把codeql文件夹放入～/CodeQL。

为了方便测试我们需要把ql可执行程序加入到环境变量当中：

```bash
export PATH=/Home/CodeQL/codeql:$PATH
source /etc/profile
```

检验：命令行输入codeql，出现如下内容就表示引擎设置完成。

![image](https://image.3001.net/images/20210808/1628393945_610f51d921c931d6b0a03.png!small)

### SDK安装

我们使用Git下载QL语言工具包，也放入～/CodeQL文件夹。

```bash
git clone https://github.com/github/codeql.git
```

这样在～/CodeQL目录下就包含了2个文件夹，引擎文件夹(codeql)和SDK文件夹(ql)。

```bash
➜  CodeQL ls
codeql ql
```

### VSCode开发插件安装

CodeQL需要使用`Visual Studio Code`来开发和调试规则，所以我们需要在VSCode上面安装CodeQL的插件。

我们安装好`Visual Studio Code`后，在它的扩展里面搜索`codeql`, 点击安装。

![image](https://image.3001.net/images/20210808/1628393988_610f5204f4068f0182a4e.png!small)

然后我们配置一下上面我们安装的`codeql引擎`路径。

![image](https://image.3001.net/images/20210808/1628394021_610f522570e932b7c0682.png!small)

到此，我们就设置好了`CodeQL`的开发环境。

## 生成数据库

### 本地生成

```bash
codeql database create 数据库名 --language=cpp --source-root=源码路径
```

比如现在要对xxl-job这个项目进行漏洞扫描

下载项目源码并进入该目录

```bash
#下载源代码
git clone https://gitee.com/xuxueli0323/xxl-job 

#创建源码数据库：
codeql database create xxljob --language=java --command="mvn clean install" --source-root=D:\xxl-job 
codeql database create job --language=cpp 
```

**主要参数：**

- --language 要根据具体项目的编译语言指定
- --command 参数如果不指定，会使用默认的编译命令和参数
- --source-root 源码路径
- --overwrite 表示 create 的目标 database 对已有的 database 做覆盖

**--language 对应关系如下：**

| **Language**          | **Identity** |
|-----------------------|--------------|
| C/C++                 | cpp          |
| C#                    | csharp       |
| Go                    | go           |
| Java                  | java         |
| javascript/Typescript | javascript   |
| Python                | python       |

**--command 中指定的脚本的要求：**

- Confirm that there is some source code for the specified language in the project.
- For codebases written in Go, JavaScript, TypeScript, and Python, do not specify an explicit --command.
- For other languages, the --command must specify a “clean” build which compiles all the source code files without
  reusing existing build artefacts. 即如果项目中原本有任何编译产生的临时或最终文件，都需要删除，一定保证编译过程完全“
  clean ”。

如某个项目的 `build.sh`：

```bash
rm -r build && mkdir build
cd build && cmake .. && make -j
```

**成功生成：**

<img src="success_generate.png" alt="img" style="zoom:67%;" />

### 线上生成

```text
ps:适合不需要打包的如python等    
此方法只是为了生成数据库，和上述第三步一样，区别只是在线生成和本地生成。

url：https://lgtm.com/dashboard
codeql database create openjdk8u322-db  --language=java --command='make images JOBS=4'

```

1.输入链接,点击follow生成

<img src="https://pic1.zhimg.com/80/v2-6c8e25f02d85742631085531d9725ae8_1440w.webp" alt="img" style="zoom:67%;" />

2.生成完成后点击下方进入下载到本地

<img src="https://pic1.zhimg.com/80/v2-78f11fdcd0ba70fe6edd4edb31c1929c_1440w.webp" alt="img" style="zoom:67%;" />



<img src="https://pic2.zhimg.com/80/v2-5c433a305467e860e2e27cae36a3b86d_1440w.webp" alt="img" style="zoom:67%;" />

## 进行项目漏洞扫描

### VSCode

​    [主要参考](https://blog.csdn.net/huangjinjin520/article/details/124010687?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166801001416782429738534%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=166801001416782429738534&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-124010687-null-null.142^v63^control,201^v3^control_2,213^v2^t3_esquery_v2&utm_term=codeql%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90&spm=1018.2226.3001.4187)

- 在 VSCode 打开扫描规则 CodeQL libraries and queries（ql），并保存工作区（文件 ---> 工作区另存）

- VSCode 添加数据库

- 打开工作区，选择要使用的扫描 CodeQL RUN Query

  <img src="https://pic2.zhimg.com/80/v2-664623cf30a2e46178bc12dc84de3481_1440w.webp" alt="img" style="zoom:50%;" />

- 执行规则扫描

  选择具体语言的规则进行扫描，例如：java语言的规则。`ql`后缀的文件是规则扫描文件

```
Eg:  CodeQL\ql\cpp\ql\src\Security
```

<img src="process_run.png" alt="img" style="zoom: 67%;" />

​ 点击 `CodeQL:Run Queries in Selected Files` 后，弹出一个对话框，选择`Yes`；即可执行扫描操作。

​ PS：可以一次选择一条或者多条规则就行扫描；但是一次性不能超过 20 条规则。

- 结果查看

<img src="https://pic3.zhimg.com/80/v2-e6cb2034df001b709dbaeff4829ba182_1440w.webp" alt="img" style="zoom: 67%;" />

### 命令行方法

```bash
#创建数据库
codeql database create databaseName --source-root=D:/xxljob --language=java

#更新数据库
codeql database upgrade databaseName

#执行扫描规则
codeql database analyze databasePath codeql-repo/java --format=csv --output=result.csv
```

- codeql-repo/java ：java 扫描规则
- --format：结果输出格式
- --output：结果文件输出路径

# ql语言学习

## CodeQL语法

CodeQL的核心引擎是不开源的

这个核心引擎的作用之一是帮助我们把 micro-service-seclab 转换成 CodeQL 能识别的中间层数据库。

由于CodeQL开源了所有的规则和规则库部分，所以我们能够做的就是编写符合我们业务逻辑的QL规则，然后使用CodeQL引擎去跑我们的规则。

![image](https://image.3001.net/images/20210808/1628394186_610f52ca969589d7b9214.png!small)

## 注释

在使用命令行解析或vscode的时候，必须包含以下两个注释:

（参考 https://github.com/github/codeql/blob/main/docs/query-metadata-style-guide.md）

https://codeql.github.com/docs/writing-codeql-queries/introduction-to-ql/

**@kind**

定义查询的类型，可以定义的类型为以下3个

1. problem：必须两列或其倍列，结构：element,string
2. path-problem：必须包含四列，结构：element, source, sink, string，后续查询必须以element,string结构
3. metric

**@id**

定义该查询的唯一标识，应以语言为开头，支持的语言如下：

- C and C++: `cpp`
- C#: `cs`
- Go: `go`
- Java: `java`
- JavaScript and TypeScript: `js`
- Python: `py`

后面建议接上一个该查询所针对的问题，比如

- `@id cs/command-line-injection`
- `@id java/string-concatenation-in-loop`

### 语句

| Query part                                                                                                                                   | Purpose                              | Details                                               |
|----------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|-------------------------------------------------------|
| import java                                                                                                                                  | 导入标准Code                             | 每个查询语言以一个或多个import开始                                  |
| from MethodAccess methodAcces                                                                                                                | 定义查询的变量，格式如下： <type> <variable name> | 这里定义MethodAccess 类型的变量                                |
| where methodAccess.getMethod().hasName("lookup") and methodAccess.getMethod().getDeclaringType().hasQualifiedName("javax.naming", "Context") | 定义变量的条件                              | 定义methodAcces的方法名为lookup，调用该方法的类为javax.naming.Context |
| select methodAccess,methodAccess.getCaller().getName()                                                                                       | 定义匹配后结果数据                            | 返回方法执行参数以及方法名                                         |

CodeQL的查询语法类似SQL：

```sql
from [datatype] var
where condition(var = something)
select var
```

Eg:  在所有的整形数字i中，当i==1的时候，输出i

```sql
import
java  #表示我们要引入CodeQl的类库
，表示分析项目为java
，所以在ql语句里
，必不可少
 
from int i   #定义一个变量i
，它的类型是int
，表示我们获取所有的int类型的数据
where i = 1  #当i等于1的时候
，符合条件
；
select i #输出 i 
```

