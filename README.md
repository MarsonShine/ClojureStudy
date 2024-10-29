# ClojureStudy
## require 关键字

`require` 可以导入依赖包：

```clojure
(require '[clojure.string])
(clojure.string/upper-case "clojure") ;; 
```

调用包 `clojure.string` 的 `upper-case` 方法。

require 可以重命名方法：

```clojure
(require '[clojure.string :as str])
(str/upper-case "clojure")
```

这样就不用写完整的包名。

还有更简单的，通过 require + refer 导入并申明：

```clojure
(require '[clojure.string :refer [upper-case]])
(upper-case "clojure")
```

## doc 查看文档

可以通过评估 (`doc MY-VAR-NAME`)来打印给定 Var 的 API 文档：

```clojure
user=> (doc nil?)
-------------------------
clojure.core/nil?
([x])
  Returns true if x is nil, false otherwise.
nil
user=> (doc clojure.string/upper-case)
-------------------------
clojure.string/upper-case
([s])
  Converts string to all upper-case.
nil
```

## source 查看源码

您还可以使用 `source` 查看用于定义 Var 的源代码：

```clojure
(source some?)
(defn some?
  "Returns true if x is not nil, false otherwise."
  {:tag Boolean
   :added "1.6"
   :static true}
  [x] (not (nil? x)))
nil
```

## dir 列出包命名空间中所有 Vars 名称

```clojure
(dir clojure.string)
blank?
capitalize
ends-with?
escape
includes?
index-of
join
...
```

## aprops

如果你不太记得某个 Var 的名字，可以使用 apropos 进行搜索：

```clojure
(apropos "index")
(clojure.core/indexed? clojure.core/keep-indexed clojure.core/map-indexed clojure.string/index-of clojure.string/last-index-of)
```

## find-doc

查询文档内容：

```clojure
(find-doc "indexed")
```

## 语法

### 符号和标识符

```clojure
map ;符号  
+ ;符号 - 允许使用大多数标点符号  
clojure.core/+ ;带命名空间的符号  
nil ;空值  
true false ;布尔值  
:alpha ;关键字  
:release/alpha ;带命名空间的关键字  
```

符号由字母、数字和其他标点组成，用于引用其他事物，比如函数、值、命名空间等。符号可以选择性地带有命名空间，与名称之间用斜杠分隔。

有三个特殊的符号被视为不同的类型：`nil` 表示空值，`true` 和 `false` 表示布尔值。

关键字以冒号开头，并且始终自我求值。它们在 Clojure 中常用于枚举值或属性名称。

### 字面量集合

Clojure 还为四种集合类型提供了字面量语法：

```clojure
'(1 2 3) ;列表 
[1 2 3] ;向量 
#{1 2 3} ;集合
{:a 1, :b 2} ;映射  
```

我们稍后会详细讨论这些内容——目前你只需知道这四种数据结构可用于创建复合数据。

### Clojure 求值

在 Clojure 中，源代码由读取器（Reader）作为字符读取。读取器可以从 `.clj` 文件读取源代码，也可以交互式地接收一系列表达式。读取器输出 Clojure 数据，而 Clojure 编译器则将这些数据转化为 JVM 的字节码。

这里有两个关键点：

1. 源代码的基本单位是 **Clojure 表达式**，而不是 Clojure 源文件。源文件会被读取为一系列表达式，就像在 REPL 中交互式输入这些表达式一样。

2. 将读取器和编译器分离是一个重要的设计，使宏有了发挥空间。宏是特殊的函数，它们接收代码（作为数据）并输出代码（作为数据）。你是否能看到在评估模型中可以插入一个用于宏展开的循环位置？

#### 结构与语义

考虑以下 Clojure 表达式：

![](./asserts/structure-and-semantics.png)

此图展示了语法（绿色，表示读取器生成的 Clojure 数据结构）和语义（蓝色，表示 Clojure 运行时对数据的理解）之间的区别。

大多数字面量的 Clojure 形式会自我求值，除了符号和列表。符号用于引用其他内容，在求值时返回它们所引用的对象。列表（如图所示）被求值为调用操作。

在图中，`(+ 3 4)` 被读取为一个包含符号 `+` 和两个数字 `3` 和 `4` 的列表。列表的第一个元素（即 `+` 的位置）称为“函数位置”，用于找到要调用的对象。尽管函数是最明显的调用对象，Clojure 运行时还识别一些特殊操作符、宏以及少量其他可调用对象。

考虑上述表达式的求值过程：

- `3` 和 `4` 自我求值为长整型数字（`long`）
- `+` 求值为实现加法的函数
- 求值列表将调用 `+` 函数，并以 `3` 和 `4` 作为参数

许多编程语言同时拥有语句和表达式，其中语句具有某些状态变化的效果，但不返回值。在 Clojure 中，一切都是表达式，并且都会求值为某个值。某些表达式（但并非大多数）也具有副作用。