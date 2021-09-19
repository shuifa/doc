# php语言的执行原理

![php执行顺序](https://oushuifa.github.io/doc/image/exec.png)

1. 词法分析将php转换成有意义的Token，此步骤的词法分析器使用 [Re2c](http://re2c.org/) 实现。
2. 语法分析器将Token和符合文法规则的代码生成抽象语法树（简称AST）。语法分析器基于 [Bison](https://www.gnu.org/software/bison/) 实现。语法分析使用了BNF（Backus-NaurForm,巴科斯范式）来表达文法规则，Bison借助状态机、
状态转移表和压栈、出栈等一系列操作，生成抽象语法树。
3. 上步的抽象语法树生成对应的opcode，并被虚拟机执行。

# php内核架构

![php内核架构](https://oushuifa.github.io/doc/image/arch.png)

php架构大致分为四个部分。
1. **Zend 引擎** ：