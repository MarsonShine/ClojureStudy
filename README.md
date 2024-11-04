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
{:a 1, :b 2} ;字典  
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

现在，让我们考虑一下如何在 Clojure 中交互式地求值表达式。

#### 使用单引用延迟求值

有时，延迟求值是有用的，特别是对符号和列表而言。有些情况下，符号仅仅应该作为符号，而不去查找其引用的内容：

```clojure
user=> 'x
x
```

有时，列表也仅仅是一个数据值的列表（而不是需要求值的代码）：

```clojure
user=> '(1 2 3)
(1 2 3)
```

一个常见的错误是意外地将数据列表当作代码求值：

```clojure
user=> (1 2 3)
Execution error (ClassCastException) at user/eval156 (REPL:1).
class java.lang.Long cannot be cast to class clojure.lang.IFn
```

暂时不必过于担心引用（`quote`），但在这些内容中，你会偶尔看到它用于避免对符号或列表的求值。

```
user=>(+ 3 4)
7
user=>(+ 10 *1)
17
user=>(+ *1 *2)
24
```

- `*1`：最后一个结果。
- `*2`：两个表达式前的结果。
- `*3`：三个表达式前的结果。

### Clojure 基础

#### def

当你在 REPL 中求值时，保存一些数据以供后用是很有用的。我们可以使用 `def` 来实现：

```clojure
user=> (def x 7)
#'user/x
```

`def` 是一种特殊形式，它在当前命名空间中将符号（`x`）与一个值（`7`）关联起来。这种关联称为“变量”（var）。在实际的 Clojure 代码中，变量通常应该引用一个常量值或一个函数，但在 REPL 中为了方便，通常会反复定义和重新定义它们。

注意上面的返回值是 `#'user/x`——这是一个变量的字面表示：`#'` 后跟带命名空间的符号。`user` 是默认的命名空间。

记住，符号通过查找它们所引用的内容来求值，因此我们只需使用该符号即可取回它的值：

```clojure
user=> (+ x x)
14
```

#### Printing

学习一门语言时，最常做的事情之一就是打印值。Clojure 提供了几种打印值的函数：

|        | 适用于人类阅读 | 可作为数据读取 |
| ------ | -------------- | -------------- |
| 换行   | println        | prn            |
| 不换行 | print          | pr             |

适合人类阅读的形式会将特殊字符（如换行和制表符）转为其打印形式，并省略字符串中的引号。`println` 常用于调试函数或在 REPL 中打印值。`println` 可以接收任意数量的参数，并在每个参数的打印值之间加上空格：

```
user=> (println "What is this:" (+ 1 2))
What is this: 3
```

`println` 函数有副作用（打印），其返回结果为 `nil`。

注意上面 `"What is this:"` 没有打印出引号，这并不是读取器可以再次读取的数据形式。

为此，可以使用 `prn` 以数据形式打印：

```
user=> (prn "one\n\ttwo")
"one\n\ttwo"
```

此时，打印的结果是一个有效形式，读取器可以再次读取。根据上下文，你可以选择更适合人类阅读的形式或数据形式。

### 测试题

1.Using the REPL, compute the sum of 7654 and 1234.

```
user=> (+ 7654 1234)
```

2.Rewrite the following algebraic expression as a Clojure expression: ( 7 + 3 * 4 + 5 ) / 10.

```
user=>(/ (+ (+ (* 3 4) 7) 5) 10)
```

3.Using REPL documentation functions, find the documentation for the `rem` and `mod` functions. Compare the results of the provided expressions based on the documentation.

```
user=> doc rem
doc mod
```

4.Using `find-doc`, find the function that prints the stack trace of the most recent REPL exception.

```
(find-doc "stack")
```

## 函数

### 创建函数

Clojure 是一门函数式语言。函数是第一类对象，可以作为参数传递给其他函数或从其他函数返回。大多数 Clojure 代码主要由纯函数（无副作用）组成，因此相同的输入总是产生相同的输出。

`defn` 用于定义一个命名函数：

```clojure
;;    name   params         body
;;    -----  ------  -------------------
(defn greet  [name]  (str "Hello, " name) )
```

此函数只有一个参数 `name`，不过你可以在参数向量中包含任意数量的参数。

调用函数时，将函数名放在“函数位置”（列表的第一个元素）即可：

```clojure
user=> (greet "students")
"Hello, students"
```

#### 多参函数

函数可以定义为接受不同数量的参数（不同“参数数量”）。不同的参数数量必须在同一个 `defn` 中定义——如果多次使用 `defn` 定义相同函数名，会覆盖前一个定义。

每种参数数量的定义格式为 `([参数*] 表达式*)`。一个参数数量的定义可以调用另一个。函数体可以包含任意数量的表达式，返回值是最后一个表达式的结果。

```
(defn messenger
  ([]     (messenger "Hello world!"))
  ([msg]  (println msg)))
```

例如，这个函数声明了两个参数数量（0个参数和1个参数）。0参数的形式会调用1参数形式，并使用一个默认值进行打印。我们通过传入相应数量的参数来调用这些函数：

```clojure
user=> (messenger)
Hello world!
nil

user=> (messenger "Hello class!")
Hello class!
nil
```

> 类似于 C# 中的函数重载

#### 可变参数函数

函数还可以定义为接受不定数量的参数——这称为“可变参数”函数。可变参数必须放在参数列表的末尾，并会被收集到一个序列中供函数使用。

可变参数的起始由 `&` 标记。

```clojure
(defn hello [greeting & who]
  (println greeting who))
```

例如，此函数接收一个参数 `greeting` 和不定数量的参数（0个或更多），这些可变参数将被收集到名为 `who` 的列表中。我们可以通过传入三个参数来调用它：

```clojure
user=> (hello "Hello" "world" "class")
Hello (world class)
```

可以看到，当 `println` 打印 `who` 时，它被打印为一个包含两个元素的列表。

#### 匿名函数

匿名函数可以通过 `fn` 创建：

```clojure
;;    params         body
;;   ---------  -----------------
(fn  [message]  (println message) )
```

由于匿名函数没有名称，因此无法在以后引用它。通常，匿名函数会在将其传递给另一个函数时创建。

也可以立即调用它（这种用法不常见）：

```clojure
;;     operation (function)             argument
;; --------------------------------  --------------
(  (fn [message] (println message))  "Hello world!" )

;; Hello world!
```

在这里，我们将匿名函数定义在一个更大表达式的函数位置，并直接用参数调用它。

在许多编程语言中，语句用于执行操作但不返回值，而表达式则返回值。而在 Clojure 中，所有代码块都是表达式，都会返回一个值，即使是像 `if` 这样的流程控制语句。

##### defn vs fn

可以将 `defn` 看作是 `def` 和 `fn` 的结合体。`fn` 定义了一个函数，而 `def` 将这个函数绑定到一个名称上。以下两种写法是等效的：

```clojure
(defn greet [name] (str "Hello, " name))

(def greet (fn [name] (str "Hello, " name)))
```

#### 匿名函数语法

Clojure 读取器实现了一种更简洁的匿名函数语法：`#()`。这种语法省略了参数列表，并根据参数的位置为它们命名。

- `%` 用于单个参数
- `%1`、`%2`、`%3` 等用于多个参数
- `%&` 用于剩余的（可变）参数

由于嵌套匿名函数会导致参数命名的歧义，因此不允许嵌套使用这种语法。

```clojure
;; Equivalent to: (fn [x] (+ 6 x))
#(+ 6 %)

;; Equivalent to: (fn [x y] (+ x y))
#(+ %1 %2)

;; Equivalent to: (fn [x y & zs] (println x y zs))
#(println %1 %2 %&)
```

#### 陷阱

一个常见需求是创建一个匿名函数，它接收一个元素并将其包装到一个向量中。你可能会尝试这样写：

```clojure
;; DO NOT DO THIS
#([%])
```

这个匿名函数实际展开后等效于：

```clojure
(fn [x] ([x]))
```

> 在这里：
>
> - `([x])` 的意思是：构建一个包含 `x` 的向量，并尝试调用它。然而，在 Clojure 中，`([x])` 不是合法的写法，因为 **`([x])` 意图调用一个向量**，但向量不能作为函数被调用。
> - 因此，这种写法会导致程序出错。

这种形式会将元素包装到一个向量中，但也会尝试用一个空参数调用该向量（多出来的一对圆括号）。正确的写法是：

```clojure
;; Instead do this:
#(vector %)

;; or this:
(fn [x] [x])

;; or most simply just the vector function itself:
vector
```

### 函数调用

#### apply

`apply` 函数用于调用一个函数，并传入0个或多个固定参数，同时从最后的序列中获取剩余所需的参数。最后一个参数必须是一个序列。

```clojure
(apply f '(1 2 3 4))    ;; same as  (f 1 2 3 4)
(apply f 1 '(2 3 4))    ;; same as  (f 1 2 3 4)
(apply f 1 2 '(3 4))    ;; same as  (f 1 2 3 4)
(apply f 1 2 3 '(4))    ;; same as  (f 1 2 3 4)
```

以下四种调用都等效于 `(f 1 2 3 4)`。当参数以序列形式提供但必须以序列中的值调用函数时，`apply` 非常有用。

例如，你可以使用 `apply` 来避免编写如下代码：

```clojure
(defn plot [shape coords]   ;; coords is [x y]
  (plotxy shape (first coords) (second coords)))
```

相反，你可以简单地写成：

```clojure
(defn plot [shape coords]
  (apply plotxy shape coords))
```

### 本地变量和闭包

#### let

`let` 在“词法作用域”内将符号绑定到值。词法作用域会创建一个新的名称上下文，嵌套在周围的上下文中。在 `let` 中定义的名称会优先于外部上下文中的同名符号。

```clojure
;;      bindings     name is defined here
;;    ------------  ----------------------
(let  [name value]  (code that uses name))
```

每个 `let` 表达式可以定义0个或多个绑定，并在主体中包含0个或多个表达式。

```clojure
(let [x 1
      y 2]
  (+ x y))
```

此 `let` 表达式为 `x` 和 `y` 创建了两个本地绑定。表达式 `(+ x y)` 处于 `let` 的词法作用域中，因此 `x` 被解析为1，`y` 被解析为2。在 `let` 表达式之外，`x` 和 `y` 将不再具有意义，除非它们已在外部绑定到某个值。

```clojure
(defn messenger [msg]
  (let [a 7
        b 5
        c (clojure.string/capitalize msg)]
    (println a b c)
  ) ;; end of let scope
) ;; end of function
```

`messenger` 函数接受一个 `msg` 参数。这里 `defn` 也为 `msg` 创建了词法作用域——它仅在 `messenger` 函数内有意义。

在该函数作用域内，`let` 创建了一个新作用域来定义 `a`、`b` 和 `c`。如果在 `let` 表达式之后尝试使用 `a`，编译器会报错。

#### 闭包

`fn` 特殊形式会创建一个“闭包”。它“闭合”了周围的词法作用域（例如上述的 `msg`、`a`、`b` 或 `c`），并在词法作用域之外捕获这些值。

```clojure
(defn messenger-builder [greeting]
  (fn [who] (println greeting who))) ; closes over greeting

;; greeting provided here, then goes out of scope
(def hello-er (messenger-builder "Hello"))

;; greeting value still available because hello-er is a closure
(hello-er "world!")
;; Hello world!
```

### Java 交互

#### 调用 Java 代码

以下是从 Clojure 调用 Java 的调用约定摘要：

| Task     | Java                | Clojure            |      |
| :------- | :------------------ | :----------------- | :--- |
| 实例化   | `new Widget("foo")` | `(Widget. "foo")`  |      |
| 实例方法 | `rnd.nextInt()`     | `(.nextInt rnd)`   |      |
| 实例字段 | `object.field`      | `(.-field object)` |      |
| 静态方法 | `Math.sqrt(25)`     | `(Math/sqrt 25)`   |      |
| 静态字段 | `Math.PI`           | `Math/PI`          |      |

#### Java 方法 vs 函数

- Java 的方法不是 Clojure 的函数。
- 作为参数无法存储或传递它们。
- 在必要时能在函数中包装。

```clojure
;; make a function to invoke .length on arg
(fn [obj] (.length obj))

;; same thing
#(.length %)
```

### 练习题

1. 定义一个不带参数并打印 "Hello "的函数 greet。 用实现替换______： (`defn greet [] _`)

   ```clojure
   (def greet []
     (println "Hello"))
   ```

2. 使用 def 重新定义 greet，首先使用 fn 特殊形式，然后使用 #() 阅读器宏。

   ```clojure
   (def greet 
     (fn [] (printn "Hello")))
     
   (def greet
     #(println "Hello"))
   ```

3. 顶一个 greeting 函数，要求：

   - 无参，返回 "Hello, World!"。
   - 1个参数，返回 "Hello, x!"。
   - 2个参数 x,y，返回 "x, y!"。

   ```clojure
   (def greeting
     ([] (println "Hello, World!"))
     ([x] (println "Hello, " x))
     ([x y] (println x "," y "!"))
   ```

4. 定义一个 `do-nothing` 函数，传递1个参数 x 并返回它。

   ```clojure
   (defn do-nothing [x] x)
   ```

   

## 序列集合

Clojure 集合将值“集合”成复合值。 Clojure 有四种主要的集合类型：向量、列表、集合和字典。在这四种集合类型中，向量和列表是有序的。

### 向量

向量是索引的、顺序的数据结构。向量使用 `[ ]` 表示，如下所示：

```
[1 2 3]
```

#### 索引访问

“索引”意味着可以通过索引来检索向量中的元素。在Clojure（与 Java 类似）中，索引从 0 开始，而不是 1。使用 `get` 函数可以在给定索引处检索元素：

```clojure
user=> (get ["abc" false 99] 0)
"abc"
user=> (get ["abc" false 99] 1)
false
```

使用无效索引值调用 get 会返回 nil：

```clojure
user=> (get ["abc" false 99] 14)
nil
```

#### count

Clojure 所有的集合都能计数：

```clojure
user=> (count [1 2 3])
3
```

#### 构造

除了字面量 `[ ]` 语法，还可以使用 `vector` 函数创建 Clojure 向量：

```clojure
user=> (vector 1 2 3)
[1 2 3]
```

#### 添加元素

元素用 `conj`（conjoin 的缩写）添加到向量中。 元素总是添加到向量的末尾：

```clojure
user=> (def v [1 2 3])
#'user/v
user=> (conj v 4 5 6)
[1 2 3 4 5 6]
```

`conj` 返回一个新的向量集合，如果我们检查一下原始向量，就会发现它没有变化：

```clojure
user=> v
[1 2 3]
```

任何“更改”集合的函数都会返回一个新实例。 您的程序需要记住或传递更改后的实例，以便利用它。

### 列表

列表是顺序的链表，新增元素时会添加到列表的头部，而不像向量那样添加到尾部。

#### 构造

由于列表在求值时会将第一个元素作为函数调用，我们必须对列表进行引用以防止求值：

```clojure
(def cards '(10 :ace :jack 9))
```

> 单引号（`'`）是**引用操作符**，用于告诉编译器或解释器不要对后面的表达式进行求值，而是将其作为一个**原始数据结构**（通常是列表）对待。
>
> 这样，`cards` 被定义为一个包含四个元素的列表。

列表没有索引，因此需要使用 `first` 和 `rest` 来遍历。

```clojure
user=> (first cards)
10
user=> (rest cards)
'(:ace :jack 9)
```

#### 添加元素

`conj` 可以像处理向量一样用于向列表中添加元素。然而，`conj` 总是将元素添加到可以在数据结构中保持常量时间的位置。对于列表，元素会添加到前面：

```clojure
user=> (conj cards :queen)
(:queen 10 :ace :jack 9)
```

#### 栈访问

列表还可以用作栈，使用 `peek` 和 `pop` 操作：

```clojure
user=> (def stack '(:a :b))
#'user/stack
user=> (peek stack)
:a
user=> (pop stack)
(:b)
```

### 哈希集合

如前一节所述，Clojure 有四种主要的集合类型：向量、列表、集合和字典。在这四种集合类型中，集合和字典是哈希集合，旨在高效地查找元素。

#### Sets
集合类似于数学中的集合——无序且不包含重复元素。集合非常适合于高效地检查集合中是否包含某个元素，或用于删除任意元素。

```clojure
(def players #{"Alice", "Bob", "Kelly"})
```

#### 添加元素

作为向量和列表，`conj` 用来 添加元素的。

```clojure
user=> (conj players "Fred")
#{"Alice" "Fred" "Bob" "Kelly"}
```

#### 移除元素

`disj`(disjoin) 函数用来删除集合中的一个或多个元素。

```clojure
user=> players
#{"Alice" "Kelly" "Bob"}
user=> (disj players "Bob" "Sal")
#{"Alice" "Kelly"}
```

正如您所看到的，`disj` 可以将集合中不存在的元素去掉。

#### 检查元素是否存在

```clojure
user=> (contains? players "Kelly")
true
```

#### 排序集合

排序集根据比较器函数进行排序，该函数可以比较两个元素。 默认情况下，使用 Clojure 的比较函数，对数字、字符串等按“自然”顺序排序。

```clojure
user=> (conj (sorted-set) "Bravo" "Charlie" "Sigma" "Alpha")
#{"Alpha" "Bravo" "Charlie" "Sigma"}
```

一个自定义比较器能通过 `sorted-set-by`

#### into

`into` 函数用来将一个集合转入到另一个集合。

```clojure
user=> (def players #{"Alice" "Bob" "Kelly"})
user=> (def new-players ["Tim" "Sue" "Greg"])
user=> (into players new-players)
#{"Alice" "Greg" "Sue" "Bob" "Tim" "Kelly"}
```

`into` 返回与其第一个参数类型相同的集合。

### 字典

字典通常用于两种情况——管理键与值的关联以及表示领域应用数据。在其他语言中，第一个用例通常被称为字典或哈希字典。

#### 创建字面量字典

字典通过 `{}` 包围的交替键值对来表示：

```clojure
(def scores {"Fred"  1400
             "Bob"   1240
             "Angela" 1024})
```

在 REPL 中打印字典时，Clojure 会在每个键/值对之间插入逗号。这仅用于提高可读性——在 Clojure 中，逗号被视为空白字符。你可以在有助于可读性的情况下随意使用它们！

```clojure
;; same as the last one!
(def scores {"Fred" 1400, "Bob" 1240, "Angela" 1024})
```

#### 添加新的键值对

`assoc` 函数用于添加新的键值对到字典：

```clojure
user=> (assoc scores "Sally" 0)
{"Angela" 1024, "Bob" 1240, "Fred" 1400, "Sally" 0}
```

如果 `assoc` 键存在，则值会被替代。

```clojure
user=> (assoc scores "Bob" 0)
{"Angela" 1024, "Bob" 0, "Fred" 1400}
```

#### 删除键值对

删除键值对的补充操作是 `dissoc`（"dissociate"）：

```clojure
user=> (dissoc scores "Bob")
{"Angela" 1024, "Fred" 1400}
```

#### 按键查询

这里有一些方法在字段中查询值。最常用是 `get` 函数：

```clojure
user=> (get scores "Angela")
1024
```

当相关字典被视为常量查找表时，通常会调用字典本身，将其视为一个函数：

```clojure
user=> (def directions {:north 0
                        :east 1
                        :south 2
                        :west 3})
#'user/directions

user=> (directions :north)
0
```

除非能保证字典结果不为零，否则不应直接调用字典：

```clojure
user=> (def bad-lookup-map nil)
#'user/bad-lookup-map

user=> (bad-lookup-map :foo)
Execution error (NullPointerException) at user/eval154 (REPL:1).
null
```

#### 查找默认值

如果要进行查找，并在未找到键时返回默认值，请将默认值指定为一个额外参数：

```clojure
user=> (get scores "Sam" 0)
0

user=> (directions :northwest -1)
-1
```

使用默认值还有助于区分缺失的键和值为零的现有键。

#### 检查包含

还有两个函数可以帮助检查地图是否包含条目。

```clojure
user=> (contains? scores "Fred")
true

user=> (find scores "Fred")
["Fred" 1400]
```

`contains` 函数表示目标字典集指定键是否存在。`find` 函数是在字典集查找键值对条目，而不是仅仅是值。

#### Keys 还是 Values

你也可以只获得字典中的所有键和值：

```clojure
user=> (keys scores)
("Fred" "Bob" "Angela")

user=> (vals scores)
(1400 1240 1024)
```

虽然字典是无序的，但可以保证按“序列”顺序走的 keys、vals 和其他函数总是以相同的顺序走特定的字典实例条目。

#### 构建字典

`zipmap` 函数可以用来将两个序列压缩到一个字典中：

```clojure
user=> (def players #{"Alice" "Bob" "Kelly"})
#'user/players

user=> (zipmap players (repeat 0))
{"Kelly" 0, "Bob" 0, "Alice" 0}
```

使用 Clojure 的序列函数（我们尚未讨论），还有其他多种方法来构建字典。 稍后再讨论这些方法！

```clojure
;; with map and into
(into {} (map (fn [player] [player 0]) players))

;; with reduce
(reduce (fn [m player]
          (assoc m player 0))
        {} ; initial value
        players)
```

#### 组合字典

`merge` 函数可用于将多个字典合并为一个字典：

```clojure
user=> (def new-scores {"Angela" 300 "Jeff" 900})
#'user/new-scores

user=> (merge scores new-scores)
{"Fred" 1400, "Bob" 1240, "Jeff" 900, "Angela" 300}
```

我们在这里合并了两个字典，但你也可以传入更多字典。

如果两个字典包含相同的键，则以最右边的字典为准。此外，你可以使用 `merge-with` 来提供一个函数，当发生冲突时调用该函数：

```clojure
user=> (def new-scores {"Fred" 550 "Angela" 900 "Sam" 1000})
#'user/new-scores

user=> (merge-with + scores new-scores)
{"Sam" 1000, "Fred" 1950, "Bob" 1240, "Angela" 1924}
```

在发生冲突时，该函数会作用于两个值，以获得新的值。

#### 排序字典

与排序集类似，排序字典基于比较器，使用比较作为默认比较器函数，保持键的排序顺序。

```clojure
user=> (def sm (sorted-map
         "Bravo" 204
         "Alfa" 35
         "Sigma" 99
         "Charlie" 100))
{"Alfa" 35, "Bravo" 204, "Charlie" 100, "Sigma" 99}

user=> (keys sm)
("Alfa" "Bravo" "Charlie" "Sigma")

user=> (vals sm)
(35 204 100 99)
```

### 表示应用域信息

当我们需要用事先知道的同一组字段来表示许多域信息时，可以使用带关键字键的映射。

```clojure
(def person
  {:first-name "Kelly"
   :last-name "Keen"
   :age 32
   :occupation "Programmer"})
```

#### 字段访问

由于这是一个映射，我们已经讨论过的通过键查找值的方法也同样适用：

```clojure
user=> (get person :occupation)
"Programmer"

user=> (person :occupation)
"Programmer"
```

但实际上，获取字段值的最常用方法是调用关键字。与映射和集合一样，关键字也可以是函数。调用关键字时，它会在传递给它的关联数据结构中查找自己。

```clojure
user=> (:occupation person)
"Programmer"
```

关键字调用还可以选择默认值：

```clojure
user=> (:favorite-color person "beige")
"beige"
```

#### 更新字段

由于这是一个字典，我们可以使用 `assoc` 来添加或修改字段：

```clojure
user=> (assoc person :occupation "Baker")
{:age 32, :last-name "Keen", :first-name "Kelly", :occupation "Baker"}
```

#### 删除字段

使用 `dissoc` 删除字段：

```clojure
user=> (dissoc person :age)
{:last-name "Keen", :first-name "Kelly", :occupation "Programmer"}
```

#### 实体嵌套

实体嵌套在其他实体中的情况很常见：

```clojure
(def company
  {:name "WidgetCo"
   :address {:street "123 Main St"
             :city "Springfield"
             :state "IL"}})
```

您可以使用 `get-in` 访问嵌套实体内任何层级的字段：

```clojure
user=> (get-in company [:address :city])
"Springfield"
```

您也可以使用 `assoc-in` 或 `update-in` 来修改嵌套实体：

```clojure
user=> (assoc-in company [:address :street] "303 Broadway")
{:name "WidgetCo",
 :address
 {:state "IL",
  :city "Springfield",
  :street "303 Broadway"}}
```

#### Record

使用映射的另一种选择是创建“记录”（record）。记录专门为此类用例设计，通常具有更好的性能。此外，记录有一个命名的“类型”，可用于多态行为（关于多态稍后会详细介绍）。

> **`record`（记录）** 是一种用于定义具有固定字段的**数据结构**。
>
> `record` 是不可变的，具有良好的性能特性。

Records 在定义时包含 record 实例的字段名称列表。这些字段将在每个 record 实例中被视为关键字键。

```clojure
;; Define a record structure
(defrecord Person [first-name last-name age occupation])

;; Positional constructor - generated
(def kelly (->Person "Kelly" "Keen" 32 "Programmer"))

;; Map constructor - generated
(def kelly (map->Person
             {:first-name "Kelly"
              :last-name "Keen"
              :age 32
              :occupation "Programmer"}))
```

> **`defrecord`**：用于定义一个新的记录类型。
>
> **`Person`**：记录的名称，相当于定义了一个新的类型。
>
> **`[first-name last-name age occupation]`**：字段列表，指定了 `Person` 记录包含的字段。
>
> 定义完成后，Clojure 会自动生成一些构造函数和方法，方便你创建和操作 `Person` 对象。
>
> (def kelly (->Person "Kelly" "Keen" 32 "Programmer"))：使用 `defrecord` 位置函数初始化 record 实例。

记录的使用方法与字典几乎完全相同，但需要注意的是，记录不能像字典一样作为函数调用。

```clojure
user=> (:occupation kelly)
"Programmer"
```

## 流程控制

### 语句 vs 表达式

Java 表达式返回值，这是语句做不到的。

```java
// "if" is a statement because it doesn't return a value:
String s;
if (x > 10) {
    s = "greater";
} else {
    s = "less or equal";
}
obj.someMethod(s);

// Ternary operator is an expression; it returns a value:
obj.someMethod(x > 10 ? "greater" : "less or equal");
```

在 Clojure 中，一切都是表达式！每个表达式都会返回一个值，而包含多个表达式的代码块会返回最后一个表达式的值。那些仅执行副作用的表达式返回 `nil`。

### 流程控制表达式
因此，流程控制操作符也是表达式！

流程控制操作符是可组合的，因此可以在任何地方使用它们。这减少了重复代码和中间变量的使用。

此外，通过宏，流程控制操作符也是可扩展的，这允许用户代码扩展编译器。我们今天不会讨论宏，但你可以 [Macros](https://clojure.org/reference/macros), [Clojure from the Ground Up](https://aphyr.com/posts/305-clojure-from-the-ground-up-macros) 或 [Clojure for the Brave and True](http://www.braveclojure.com/writing-macros/) 等资料中了解更多。

#### if

`if` 是最重要的条件表达式——它包含一个条件、一个“then”分支和一个“else”分支。`if` 只会对条件选择的分支进行求值。

```clojure
user=> (str "2 is " (if (even? 2) "even" "odd"))
2 is even

user=> (if (true? false) "impossible!") ;; else is optional
nil
```

#### Truth

在 Clojure 中，所有值在逻辑上都是 true 或 false。 唯一的“假”值是 `false` 和 `nil`，所有其他值在逻辑上都是真。

```clojure
user=> (if true :truthy :falsey)
:truthy
user=> (if (Object.) :truthy :falsey) ; objects are true
:truthy
user=> (if [] :truthy :falsey) ; empty collections are true
:truthy
user=> (if 0 :truthy :falsey) ; zero is true
:truthy
user=> (if false :truthy :falsey)
:falsey
user=> (if nil :truthy :falsey)
:falsey
```

#### if 和 do

`if` 语句只接受一个“then”和“else”的单一表达式。使用 `do` 可以创建更大的代码块，使其成为单一表达式。

需要注意的是，只有当你的表达式主体有副作用时才需要这样做！（为什么？）

```clojure
(if (even? 5)
  (do (println "even")
      true)
  (do (println "odd")
      false))
```

#### when
`when` 是只有“then”分支的 `if`。它检查一个条件，然后将主体中的任意数量的表达式求值（因此不需要 `do`）。返回值是最后一个表达式的结果。如果条件为假，则返回 `nil`。

`when` 向读者传达的是没有“else”分支的语义。

```clojure
(when (neg? x)
  (throw (RuntimeException. (str "x must be positive: " x))))
```

#### cond
`cond` 是一系列的测试和表达式。每个测试按顺序求值，第一个为真的测试对应的表达式会被求值并返回。

```clojure
(let [x 5]
  (cond
    (< x 2) "x is less than 2"
    (< x 10) "x is less than 10"))
```

#### cond 和 :else
如果没有测试满足条件，返回 `nil`。一种常见的惯用法是使用一个最终的 `:else` 测试。关键字（如 `:else`）总是求值为 `true`，因此将被作为默认选择。

```clojure
(let [x 11]
  (cond
    (< x 2)  "x is less than 2"
    (< x 10) "x is less than 10"
    :else  "x is greater than or equal to 10"))
```

#### case
`case` 将一个参数与一系列值进行比较以找到匹配项。这是以常量时间（而不是线性时间）完成的！不过，每个值必须是编译时的字面量（如数字、字符串、关键字等）。

与 `cond` 不同，如果没有匹配项，`case` 会抛出异常。

```clojure
user=> (defn foo [x]
         (case x
           5 "x is 5"
           10 "x is 10"))
#'user/foo

user=> (foo 10)
x is 10

user=> (foo 11)
IllegalArgumentException No matching clause: 11
```

#### 带有 `else` 表达式的 case
`case` 可以有一个最终的备用表达式，如果没有测试匹配，这个表达式将被求值。

```clojure
user=> (defn foo [x]
         (case x
           5 "x is 5"
           10 "x is 10"
           "x isn't 5 or 10"))
#'user/foo

user=> (foo 11)
x isn't 5 or 10
```

### 迭代的副作用

#### dotimes

- n 次表达式求值
- 返回 nil

```clojure
user=> (dotimes [i 3]
         (println i))
0
1
2
nil
```

#### doseq

- 对序列进行迭代
- 如果是懒序列，则强制求值
- 返回 nil

```clojure
user=> (doseq [n (range 3)]
         (println n))
0
1
2
nil
```

#### 多绑定的 doseq

- 类似于嵌套的 foreach 循环
- 处理序列内容的所有排列组合
- 返回 nil

```clojure
user=> (doseq [letter [:a :b]
               number (range 3)] ; list of 0, 1, 2
         (prn [letter number]))
[:a 0]
[:a 1]
[:a 2]
[:b 0]
[:b 1]
[:b 2]
nil
```

### Clojure 的 for

- 列表推导，而非传统 for 循环
- 用于序列排列的生成器函数
- 绑定行为类似于 doseq

```clojure
user=> (for [letter [:a :b]
             number (range 3)] ; list of 0, 1, 2
         [letter number])
([:a 0] [:a 1] [:a 2] [:b 0] [:b 1] [:b 2])
```

### 递归

#### 递归和迭代

- Clojure 提供了 `recur` 和序列抽象
- `recur` 是“经典”的递归形式
  - 使用者无法控制 `recur`，它被认为是一种较低级的工具
- 序列表示迭代为值
  - 使用者可以部分地进行迭代
- `reducers` 表示迭代为函数组合
  - 在 Clojure 1.5 中引入，这里不作详细介绍

#### `loop` 和 `recur`

- 函数式的循环结构
  - `loop` 用于定义绑定
  - `recur` 重新执行 `loop`，并带有新的绑定
- 建议优先使用高阶库函数，而不是这些低级工具。

```clojure
(loop [i 0]
  (if (< i 10)
    (recur (inc i))
    i))
```

#### defn 和 recur

- 函数参数是隐式循环绑定

```clojure
(defn increase [i]
  (if (< i 10)
    (recur (inc i))
    i))
```

#### `recur` 用于递归

- `recur` 必须位于“尾部位置”
  - 即在分支中的最后一个表达式
- `recur` 必须按位置为所有绑定的符号提供值
  - 循环绑定
  - `defn`/`fn` 的参数
- 通过 `recur` 进行的递归不会消耗堆栈。

### 异常

#### 异常处理

- 类似于 java 中的 try/catch/finally

```clojure
(try
  (/ 2 1)
  (catch ArithmeticException e
    "divide by zero")
  (finally
    (println "cleanup")))
```

#### 抛异常

```clojure
(try
  (throw (Exception. "something went wrong"))
  (catch Exception e (.getMessage e)))
```

#### 异常使用数据

- `ex-info` 传递一个消息和一个字典
- `ex-data` 获取字典
  - 或者如果未使用 `ex-info` 创建，则为 nil

```clojure
(try
  (throw (ex-info "There was a problem" {:detail 42}))
  (catch Exception e
    (prn (:detail (ex-data e)))))
```

#### with-open

```clojure
(let [f (clojure.java.io/writer "/tmp/new")]
  (try
    (.write f "some text")
    (finally
      (.close f))))

;; Can be written:
(with-open [f (clojure.java.io/writer "/tmp/new")]
  (.write f "some text"))
```

> `with-open` 是 Clojure 中的一个宏，用于**简化资源的打开和关闭操作**，特别是对于实现了 `java.io.Closeable` 接口的资源（如文件、流、读者、写者等）。它能够确保在代码块执行完毕后，无论是否发生异常，都能自动地关闭所使用的资源，避免资源泄漏。

> PS: 这个有点像 c# 的 using 语句。