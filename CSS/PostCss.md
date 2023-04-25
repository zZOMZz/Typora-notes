[toc]

# PostCss

## 一. 什么是postcss

- `PostCss`不是想`sass`和`Less`一样的预处理器(preprocessor)
  - 它不定义语法和语义, postcss并不一种语言. PostCss与Css一起工作, 可以很容易的与上述工具集成, 就是说任何有效的CSS都能被PostCss处理
- 是Css语法转换的工具
  - 它允许你自定义CSS语法只要你能保证它们能被插件识别并转换
- 是CSS生态系统中重要的一环
  - 许多有用的工具如: `Autoprefixer`, `Stylelint`, `CSSnano`都属于PostCss生态系统



## 二. PostCss工作流

<img src="C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230408181440667.png" alt="image-20230408181440667" style="zoom: 33%;" />

### 1. Parser

- 主要目的: 这个阶段(constructor)它是理解你的CSS类语法并创建它的对象表示



#### 1.1 实现parser的几种手段



##### 1. 将一个字符(string)转换为AST并写入单个文件

- 比如 [Rework analyzer](https://github.com/reworkcss/css/blob/master/lib/parse/index.js) 就是用这种方法写的. 但当代码变多时, 代码会变得难以阅读, 并且非常慢



##### 2. 将其转换为词法分析和解析两步骤(源字符串 ->令牌 -> AST)(*Split it into lexical analysis/parsing steps (source string → tokens → AST)*)

- PostCss采用这种方法, 很多Paser, 如`@babel/parser`(parser behind babel), `CSSTree`都是用这个方式实现的

- 词法分析: 将源字符串转换为令牌 ; 解析: 将令牌转换为AST
- 将标记化和解析步骤分开的主要原因是性能和抽象复杂性

具体步骤: 

1. 词法分析器会将源字符串分割为一系列令牌, 每个令牌代表一个语法单元, 如关键字, 标识符, 括号, 数字, 字符串等. 这些令牌通常是由正则表达式定义的模式匹配得到的

2. 解析器会将这些令牌组合成一棵抽象语法树(AST), 表示程序的结构和意图. AST是由节点组成的树形结构, 每个节点代表一个语法单元, 如表达式, 语句, 函数调用等. 这些节点通常是由语法规则定义的, 如EBNF
3. 将AST抽象语法树输出为一个文件, 通常是以某种格式(如JSON或XML)表示



#### 1.2 选择第二种方式的原因

- 将字符串转换为标记这个步骤比解析步骤要花费更多的事件
- 从另一方面, 从令牌到AST的转换在逻辑上更加复杂, 因此通过将词法分析和解析这两个步骤分离, 我们可以分别进行优化处理速度和增强代码的可读性; 如果只编写一个解析器, 需要同时考虑处理令牌和构建AST两个任务, 代码会变的难以优化和维护.





## 三. 核心结构(core structures)

### 1. Tokenizer(令牌转换器)

#### [`lib/tokenize.js`](https://github.com/postcss/postcss/blob/main/lib/tokenize.js)

- 也称为词法分析器, 接收CSS字符串, 并返回一列(list)tokens, 即将字符串流转换为标记流, 并将标记流传输到解析器中, 构建AST抽象语法树
- 其中tokens是一个简单的结构, 描述了语法的某个部分, 如`@`规则, 注释或单词. 它还可以包含位置信息, 以提供更具描述性的错误信息
- 每个令牌描述了CSS字符串中的一个语法单元, 包括类型, 值以及在字符串中的起始位置和结束位置. 这些令牌可以被AST构建器用来创建AST抽象语法树, 表示CSS的结构和意图

 例如:

```css
body {
    font-size: 16px;
    color: #333;
}
```

会被转换为:

```yaml
[
  { type: 'ident', value: 'body', start: { line: 1, column: 1 }, end: { line: 1, column: 4 } },
  { type: '{', value: '{', start: { line: 1, column: 6 }, end: { line: 1, column: 6 } },
  { type: 'ident', value: 'font-size', start: { line: 2, column: 3 }, end: { line: 2, column: 11 } },
  { type: ':', value: ':', start: { line: 2, column: 12 }, end: { line: 2, column: 12 } },
  { type: 'dimension', value: '16px', start: { line: 2, column: 14 }, end: { line: 2, column: 17 } },
  { type: ';', value: ';', start: { line: 2, column: 18 }, end: { line: 2, column: 18 } },
  { type: 'ident', value: 'color', start: { line: 3, column: 3 }, end: { line: 3, column: 7 } },
  { type: ':', value: ':', start: { line: 3, column: 8 }, end: { line: 3, column: 8 } },
  { type: 'hash', value: '#333', start: { line: 3, column: 10 }, end: { line: 3, column: 14 } },
  { type: ';', value: ';', start: { line: 3, column: 15 }, end: { line: 3, column: 15 } },
  { type: '}', value: '}', start: { line: 4, column: 1 }, end: { line: 4, column: 1 } }
]

```



### 2. Parser

#### [`lib/parse.js`](https://github.com/postcss/postcss/blob/main/lib/parse.js), [`lib/parser.js`](https://github.com/postcss/postcss/blob/main/lib/parser.js)

- 解析器(Parser)通过分析令牌列表来生成构建AST抽象语法树, 解析器在构建过程中也可以检查CSS中的语法错误, 并会尝试修复其中的一些信息
- 解析器不会使用源字符串进行解析构建, 因为直接操作源字符串会非常低效
- 解析器主要使用`tokenizer`提供的`nextToken`和`back`方法来获取单个或多个标记, 然后再用这些标记构造AST的部分--称为节点(Node)
- AST抽象语法树可以被后续的插件用来转换或分析CSS, 例如:
  - 插件可以在解析器处理标记是拦截标记, 并进行额外的处理或转换
  - 可以使用插件将AST转换为另一种格式, 例如JSON或XML
  - 可以使用插件在AST抽象语法树中进行查找和修改CSS中的特定节点



### 3. Processor

#### [`lib/processor.js`](https://github.com/postcss/postcss/blob/main/lib/processor.js)

- `Processor`是一种非常简单的结构, 它初始化插件并运行语法转换. 同时它还暴露了少量的公共API方法, 可以在文档中找到; 简化了PostCss的使用, 并提供一种便捷的方式来管理和应用各种插件
- `Processor`主要用于将插件集成到PostCss中, 并管理插件之间的协作. 它负责创建解析器和字符串转换器, 并将它们与插件一起组合, 以实现将CSS代码转换为抽象语法树AST并应用各种转换和优化
- 还提供了一些方便的方法来处理CSS代码, 如`process`和`processSync`方法. 这些方法可以用来处理单个CSS文件, 或在多个CSS文件之间进行批处理



### 4. Stringifier

#### [`lib/stringify.js`](https://github.com/postcss/postcss/blob/main/lib/stringify.js), [`lib/stringifier.js`](https://github.com/postcss/postcss/blob/main/lib/stringifier.js)

- `Stringifier`是一个基类, 用于将修改后的抽象语法树AST转换为纯CSS字符串. `Stringifier`会从提供的节点开始遍历AST树, 并调用相应的方法生成其原始字符串表示形式
- 最重要的方法是`Stringifier.stringify`, 它接受初始节点和分号指示器
- `Stringifier`主要用于将AST转换回CSS字符串, 以便输出到文件或传输到浏览器. 它可以处理所有类型的AST节点
- 因为`Srtingifier`是一个基类, 因此它可以通过子类进行扩展, 以添加特定于项目的转换逻辑. 因此你可以自定义一个`Stringifier`子类, 该子类生成的CSS代码格式化为特定的代码风格
  - **基类**: 面向对象编程中的一个概念, 指的是一个通用的, 抽象的类, 它可以作为其他具体类的模板或基础. 基类通常定义了一些通用的属性和方法, 可以被其他具体类继承和重写, 从而实现特定的功能





## 四. PostCss与Scss和Less预处理器间的关系

- PostCss, Sass(Scss)和Less是三种流行的CSS预处理器或后处理器, 它们都可以用于改进和扩展原生的Css语言
- 它们是三种不同的Css工具, 它们可以单独或组合使用, 以帮助开发者更加高效的编写, 优化和维护CSS代码

三者直接的关系

- PostCss是一种通用的CSS后处理器, 它可以通过插件系统对CSS代码进行转换, 优化和处理. PostCss本身不具有预处理器功能, 但可以与其他的预处理或后处理器一起使用
- Sass和Less是两种常见的CSS预处理器, 它们提供了一些扩展于原生Css语言的功能, 如变量, 嵌套规则, Mixin等, **Sass和less都需要在编译过程中将其语法转换为原生的CSS文件**, 然后才能在浏览器中使用

- PostCss是能与Sass或Less等预处理器一起使用的, 以提供更多的转换和优化功能, 例如能使用PostCss的`autoprefixer`插件作为Sass和Less生成适用于不同浏览器的Css]前缀



## 五. Sass和less处理css步骤

1. 词法分析(lexing): 将sass代码或less代码转换为一系列的tokens标记
2. 语法分析(Parsing): 将词法分析得到的Token序列, 按照语法规则组合成抽象语法树AST, 通过解析器完成
3. 编译(Compilation): 将抽象语法树AST转换为CSS代码, 将Sass中的一些Mixin, 变量等转换为对应的CSS代码, 可以通过一系列的转换器或者代码生成器来完成
4. 输出(output): 将编译后的CSS代码输出到一个文件中,或者直接输出到浏览器中



- 不同预处理器的编译过程的细节可能不同, 但基本都会遵从相同的基本原则