# Clojure 特有字符标识

此页面解释了 Clojure 语法中不易通过“谷歌”搜索到的字符。各部分没有特定的顺序，但为了方便起见，将相关项进行了分组。请参考[阅读器参考页面](https://clojure.org/reference/reader)作为 Clojure 阅读器的权威参考。本指南基于 [James Hughes](http://twitter.com/kouphax) 的[原始博客](https://yobriefca.se/blog/2014/05/19/the-weird-and-wonderful-characters-of-clojure/)文章，并在作者的许可下进行了更新和扩展。

## (...) -列表

列表是以链表形式实现的顺序异构集合。

三个值的列表：

```
(1 "two" 3.0)
```

## [...]-向量

向量是有顺序、有索引的异质集合。 索引以 0 为基础，举例说明在一个包含三个值的向量中检索索引 1 处的值：

```
user=> (get ["a" 13.7 :foo] 1)
13.7
```

## {...}-映射

映射是指定了交替键和值的异构集合：

```
user=> (keys {:a 1 :b 2})
(:a :b)
```

## #-分派字符

你会看到这个字符出现在其他字符旁边，例如 `#(` 或 `#"`。

`#` 是一个特殊字符，告诉 Clojure 阅读器（将 Clojure 源代码“读取”为 Clojure 数据的组件）如何使用读取表来解释下一个字符。尽管某些 Lisp 允许用户扩展读取表，但 Clojure [不允许](https://clojure.org/guides/faq#reader_macros)。

在语法引用中创建[生成符号](https://clojure.org/guides/weird_characters#gensym)时，`#` 也会用在符号的末尾。

> 换言之就是通过特殊符号 `#` 来告诉 Clojure 解释器要根据不同的情况解析 `#` 后面的内容。
>
> 如：
>
> **`#(`**：表示一个匿名函数，例如 `#(+ %1 %2)` 表示一个不需要名字的函数。
>
> **`#"`**：用于正则表达式，例如 `#"abc"` 表示一个正则表达式，用来匹配字符串 `"abc"`。

## #{...}-集合

`#{...}` 定义了一个集合（唯一值的集合），特别是哈希集合。 以下是等价的：

```
user=> #{1 2 3 4}
#{1 2 3 4}
user=> (hash-set 1 2 3 4)
#{1 2 3 4}
```

## #_ -弃元

#_ 告诉读者完全忽略下一个字符。

```
user=> [1 2 3 #_ 4 5]
[1 2 3 5]

user=> [1 2 3 #_4 5]
[1 2 3 5]

user=> {:a 1, #_#_ :b 2, :c 3}
{:a 1, :c 3}
```

## #"..."-正则表达式

`#"` 表示正则表达式的开始

```
user=> (re-matches #"^test$" "test")
"test"
```

## #(...)-匿名函数

`#(` 开始是内联函数定义的简短语法。 下面两段代码与之类似：

```
; anonymous function taking a single argument and printing it
(fn [line] (println line))

; anonymous function taking a single argument and printing it - shorthand
#(println %)
```

阅读器会将匿名函数扩展为函数定义，而函数的 "arity"（参数个数）是由 `%` 占位符的声明方式决定的。 请参阅 `%` 字符，了解有关 arity 的讨论。

```
user=> (macroexpand `#(println %))
(fn* [arg] (clojure.core/println arg)) ; argument names shortened for clarity
```

## #' - Var 引用

`#'` 是 var 引用，会扩展为对 `var` 函数的调用：

```
user=> (read-string "#'foo")
(var foo)
user=> (def nine 9)
#'user/nine
user=> nine
9
user=> (var nine)
#'user/nine
user=> #'nine
#'user/nine
```

使用时，它会尝试返回引用的 var。这在需要指代引用/声明而非其表示的值时非常有用。参见元数据（[^](https://clojure.org/guides/weird_characters#metadata)）讨论中的 `meta` 用法。

注意，`edn` 中不支持 var 引用。

## ##-符号值

Clojure 可以读取并打印符号值 `##Inf`、`##-Inf` 和 `##NaN`。 这些值在 edn 中也有提供。

```
user=> (/ 1.0 0.0)
##Inf
user=> (/ -1.0 0.0)
##-Inf
user=> (Math/sqrt -1.0)
##NaN
```

## #inst,#uuid,#js 等-标签字面量

标签字面量在 edn 中定义，并由 Clojure 和 ClojureScript 阅读器原生支持。 `#inst` 和 `#uuid` 标签由 edn 定义，而 `#js` 标签则由 ClojureScript 定义。 我们可以使用 Clojure 的 `read-string` 读取标签字面量（或直接使用）：

```
user=> (type #inst "2014-05-19T19:12:37.925-00:00")
java.util.Date ;; this is host dependent
user=> (read-string "#inst \"2014-05-19T19:12:37.925-00:00\"")
#inst "2014-05-19T19:12:37.925-00:00"
```

标记字面量告诉读者如何解析字面量值。 其他常见用法包括 `#uuid` 用于表达 UUID，而在 ClojureScript 世界中，标记字面量的一个极为常见的用法是 `#js`，它可用于将 ClojureScript 数据结构直接转换为 JavaScript 结构。 请注意，`#js` 不能递归转换，因此如果您有嵌套数据结构，请使用 [clj->js](https://cljs.github.io/api/cljs.core/clj-GTjs)。 

请注意，虽然 `#inst` 和 `#uuid` 在 edn 中可用，但 `#js` 不可用。

## `%`, `%n`, `%&` - 匿名函数参数

`%` 是匿名函数 `#(...)` 中的参数，例如 `#(* % %)`。

当匿名函数被展开时，它会变为一个 `fn` 形式，`%` 参数会被替换为自动生成的符号（为了便于阅读，此处使用 `arg1` 等替代）：

```clojure
user=> (macroexpand `#(println %))
(fn* [arg1] (clojure.core/println arg1))
```

在 `%` 后直接放置数字可以表示参数的位置（从1开始）。匿名函数的参数数量由最大编号的 `%` 参数决定。

```clojure
user=> (#(println %1 %2) "Hello " "Clojure")
Hello Clojure ; takes 2 args
user=> (macroexpand `#(println %1 %2))
(fn* [arg1 arg2] (clojure.core/println arg1 arg2)) ; takes 2 args

user=> (#(println %4) "Hello " "Clojure " ", Thank " "You!!")
You!! ; takes 4 args, doesn't use first 3 args
user=> (macroexpand `#(println %4))
(fn* [arg1 arg2 arg3 arg4] (clojure.core/println arg4)) ; takes 4 args doesn't use 3
```

不一定要使用所有参数，但需要按预期的顺序声明参数，以便外部调用者传入。

`%` 和 `%1` 可以互换使用：

```clojure
user=> (macroexpand `#(println % %1)) ; use both % and %1
(fn* [arg1] (clojure.core/println arg1 arg1)) ; still only takes 1 argument
```

此外，还有 `%&`，用于表示可变参数的匿名函数中的“其余”参数（在最高编号的匿名参数之后）。

```clojure
user=> (#(println %&) "Hello " "Clojure " ", Thank " "You!!")
(Hello Clojure , Thank You!! ) ; takes n args
user=> (macroexpand '#(println %&))
(fn* [& rest__11#] (println rest__11#))
```

匿名函数和 `%` 不属于 `edn` 格式。

## @ - 解引用

`@` 会扩展为对 `deref` 函数的调用，因此这两种形式是相同的：

```clojure
user=> (def x (atom 1))
#'user/x
user=> @x
1
user=> (deref x)
1
user=>
```

`@` 用于获取引用的当前值。 上面的示例使用 `@` 来获取原子的当前值，但 `@` 也可以应用于其他事物，如 `future`s、`delay`s、`promises`s 等，以强制计算并可能阻塞。 

请注意，`@` 在 edn 中不可用。

## `^` (and `#^`) - 元数据

`^` 是元数据标记。元数据是附加在 Clojure 中各种形式上的值映射（带有简写选项），提供附加信息，可用于文档、编译警告、类型提示和其他功能。

```clojure
user=> (def ^{:debug true} five 5) ; meta map with single boolean value
#'user/five
```

可以通过 `meta` 函数访问元数据，它应针对声明本身执行（而不是返回值）：

```clojure
user=> (def ^{:debug true} five 5)
#'user/five
user=> (meta #'five)
{:ns #<Namespace user>, :name five, :column 1, :debug true, :line 1, :file "NO_SOURCE_PATH"}
```

当只有一个值时，可以使用元数据的简写符号 `^:name`，这对于标志很有用，因为值会被设置为 `true`。

```clojure
user=> (def ^:debug five 5)
#'user/five
user=> (meta #'five)
{:ns #<Namespace user>, :name five, :column 1, :debug true, :line 1, :file "NO_SOURCE_PATH"}
```

`^` 的另一个用途是类型提示，用于告诉编译器值的类型，从而使其能够进行特定类型的优化，可能提高代码执行速度：

```clojure
user=> (def ^Integer five 5)
#'user/five
user=> (meta #'five)
{:ns #<Namespace user>, :name five, :column 1, :line 1, :file "NO_SOURCE_PATH", :tag java.lang.Integer}
```

在该示例中，可以看到 `:tag` 属性被设置了。

简写形式还可以叠加使用：

```clojure
user=> (def ^Integer ^:debug ^:private five 5)
#'user/five
user=> (meta #'five)
{:ns #<Namespace user>, :name five, :column 1, :private true, :debug true, :line 1, :file "NO_SOURCE_PATH", :tag java.lang.Integer}
```

最初，元数据使用 `#^` 声明，现在已弃用（但仍然有效）。后来简化为 `^`，这是大多数 Clojure 代码中使用的形式，但在旧代码中偶尔会遇到 `#^` 语法。

请注意，元数据在 `edn` 中可用，但类型提示不可用。

## ' - 引用

引用用于表示应读取下一个表格，但不对其进行求值。 阅读器将 `'` 扩展为对 `quote` 特殊表格的调用。

```clojure
user=> (1 3 4) ; fails as it tries to invoke 1 as a function

Execution error (ClassCastException) at myproject.person-names/eval230 (REPL:1).
class java.lang.Long cannot be cast to class clojure.lang.IFn

user=> '(1 3 4) ; quote
(1 3 4)

user=> (quote (1 2 3)) ; using the longer quote method
(1 2 3)
user=>
```

## : - 关键字

`:` 是关键字的指示符。关键字通常用作映射中的键，与字符串相比，它们能提供更快的比较速度和更低的内存开销（因为实例会被缓存和重复使用）。

```clojure
user=> (type :test)
clojure.lang.Keyword
```

或者，您也可以使用 `keyword` 函数从字符串创建关键字

```clojure
user=> (keyword "test")
:test
```

关键词也可以作为函数调用，在映射中作为关键字查找：

```clojure
user=> (def my-map {:one 1 :two 2})
#'user/my-map
user=> (:one my-map) ; get the value for :one by invoking it as function
1
user=> (:three my-map) ; it can safely check for missing keys
nil
user=> (:three my-map 3) ; it can return a default if specified
3
user => (get my-map :three 3) ; same as above, but using get
3
```

## :: - 自解析关键字

`::` 用于在当前命名空间中自动解析关键字。 如果没有指定限定符，它将自动解析到当前名称空间。 如果指定了限定符，则可以使用当前命名空间中的别名：

```clojure
user=> :my-keyword
:my-keyword
user=> ::my-keyword
:user/my-keyword
user=> (= ::my-keyword :my-keyword)
false
```

这在创建宏时非常有用。 如果要确保调用宏命名空间中另一个函数的宏能正确展开以调用该函数，可以使用 `::my-function` 来引用完全限定的名称。

注意，`::` 在 edn 中不可用。

## `#:` and `#::` - 命名空间映射语法

命名空间映射语法在 Clojure 1.9 中引入，用于在映射中的键或符号共享相同命名空间时，指定默认的命名空间上下文。

`#:ns` 语法指定一个完全限定的命名空间映射前缀，其中 `ns` 是命名空间的名称，该前缀位于映射的左大括号 `{` 之前。

例如，以下使用命名空间语法的映射字面量：

```clojure
#:person{:first "Han"
         :last "Solo"
         :ship #:ship{:name "Millennium Falcon"
                      :model "YT-1300f light freighter"}}
```

被视为：

```clojure
{:person/first "Han"
 :person/last "Solo"
 :person/ship {:ship/name "Millennium Falcon"
               :ship/model "YT-1300f light freighter"}}
```

注意，这些映射表示相同的对象——只是不同的语法形式。

`#::` 可以用来自动解析映射中关键字或符号键的命名空间，使用当前命名空间。

这两个示例是等效的：

```clojure
user=> (keys {:user/a 1, :user/b 2})
(:user/a :user/b)
user=> (keys #::{:a 1, :b 2})
(:user/a :user/b)
```

类似于自动解析关键字，你还可以使用 `#::alias` 通过 `ns` 语句中定义的命名空间别名进行自动解析：

```clojure
(ns rebel.core
  (:require
    [rebel.person :as p]
    [rebel.ship   :as s] ))

#::p{:first "Han"
     :last "Solo"
     :ship #::s{:name "Millennium Falcon"
                :model "YT-1300f light freighter"}}
```

同样被视为：

```clojure
{:rebel.person/first "Han"
 :rebel.person/last "Solo"
 :rebel.person/ship {:rebel.ship/name "Millennium Falcon"
                     :rebel.ship/model "YT-1300f light freighter"}}
```

最后，如果关键字是限定的，则使用给定的命名空间。 如果关键字的命名空间是 `_`，那么它将被视为没有命名空间：

```clojure
#:shape{:_/type "Square"
        :location/x 10
        :location/y 12
        :sides 4
        :width 2
        :height 2}
```

被视为：

```clojure
{:type "Square"
 :location/x 10
 :location/y 12
 :shape/sides 4
 :shape/width 2
 :shape/height 2}
```

## `/` - 命名空间分割符

`/` 可以是分割函数 `clojure.core//`，但也可以作为符号名称中的分隔符，用于分隔符号名称和命名空间限定符，例如 `my-namespace/utils`。 因此，命名空间限定符可以防止简单名称的命名冲突。

## `\` - 字面量字符

`\` 表示一个字面字符，如：

```
user=> (str \h \i)
"hi"
```

还有少量特殊字符用于表示特殊的 ASCII 字符：`\newline`、`\space`、`\tab`、`\formfeed`、`\backspace` 和 `\return`。

`\` 后还可以跟随一个 Unicode 字面量，格式为 `\uNNNN`。例如，`\u03A9` 表示字符 Ω。

## `$` - 内部类引用

用于引用 Java 中的内部类和接口。分隔容器类名称和内部类名称。

```clojure
(import (basex.core BaseXClient$EventNotifier)

(defn- build-notifier [notifier-action]
  (reify BaseXClient$EventNotifier
    (notify [this value]
      (notifier-action value))))
```

`EventNotifier` 是 `BaseXClient` 类的内部接口，而 `BaseXClient` 类是一个导入的 Java 类

## `->`, `->>`, `some->`, `cond->`, `as->` 等. - 线程宏

这些都是线程宏。详见 [Clojure 官方文档](https://clojure.org/guides/threading_macros)。

## ` - 语法引用

`` ` `` 是语法引用。语法引用类似于普通引用（用于延迟求值），但具有一些附加效果。

基本的语法引用看起来可能与普通引用相似：

```clojure
user=> (1 2 3)
Execution error (ClassCastException) at myproject.person-names/eval232 (REPL:1).
class java.lang.Long cannot be cast to class clojure.lang.IFn
user=> `(1 2 3)
(1 2 3)
```

然而，语法引用中的符号会根据当前命名空间完全解析：

```clojure
user=> (def five 5)
#'user/five
user=> `five
user/five
```

语法引用主要在宏中作为“模板”机制使用。我们可以写一个示例：

```clojure
user=> (defmacro debug [body]
  #_=>   `(let [val# ~body]
  #_=>      (println "DEBUG: " val#)
  #_=>      val#))
#'user/debug
user=> (debug (+ 2 2))
DEBUG:  4
4
```

宏是由编译器调用的函数，将代码作为数据传递。它们需要返回可以进一步编译和求值的代码（作为数据）。这个宏接受一个单一的主体表达式，并返回一个 `let` 形式，首先求值该主体，打印其值，然后返回该值。在这里，语法引用创建了一个列表，但并不对其求值。该列表实际上是代码。

参见 `~@` 和 `~`，它们是仅在语法引用中允许的额外语法。

## `~` - 取消引用

`~` 是取消引用。在语法引用中，与普通引用类似，表示在语法引用的表达式内不会进行求值。取消引用会关闭引用机制，对语法引用表达式内的表达式进行求值。

语法引用和取消引用是编写宏的重要工具。宏是在编译期间调用的函数，接受代码并返回代码。

## `~@` - 取消引用拼接

`~@` 是取消引用拼接。取消引用（`~`）会对一个形式进行求值，并将结果放入引用的结果中，而 `~@` 期望求值结果是一个集合，并将该集合的内容拼接到引用的结果中。

```clojure
user=> (def three-and-four (list 3 4))
#'user/three-and-four
user=> `(1 ~three-and-four) ; evaluates `three-and-four` and places it in the result
(1 (3 4))
user=> `(1 ~@three-and-four) ;  evaluates `three-and-four` and places its contents in the result
(1 3 4)
```

同样地，这也是编写宏的一个强大工具。

## `<symbol>#` - Gensym

符号末尾的 `#` 用于自动生成一个新符号。这在宏中非常有用，可以防止宏的特定符号泄漏到用户空间。在宏定义中，常规的 `let` 会失败：

```clojure
user=> (defmacro m [] `(let [x 1] x))
#'user/m
user=> (m)
Syntax error macroexpanding clojure.core/let at (REPL:1:1).
myproject.person-names/x - failed: simple-symbol? at: [:bindings :form :local-symbol]
  spec: :clojure.core.specs.alpha/local-name
myproject.person-names/x - failed: vector? at: [:bindings :form :seq-destructure]
  spec: :clojure.core.specs.alpha/seq-binding-form
myproject.person-names/x - failed: map? at: [:bindings :form :map-destructure]
  spec: :clojure.core.specs.alpha/map-bindings
myproject.person-names/x - failed: map? at: [:bindings :form :map-destructure]
  spec: :clojure.core.specs.alpha/map-special-binding
```

这是因为在语法引用中，符号会被完全解析，包括这里的局部绑定 `x`。

相反，可以在变量名的末尾加上 `#`，让 Clojure 生成一个唯一的（无命名空间的）符号：

```clojure
user=> (defmacro m [] `(let [x# 1] x#))
#'user/m
user=> (m)
1
user=>
```

值得注意的是，每次在单个语法引用内使用特定的 `x#`，都会使用相同的生成名称。

如果我们展开此宏，可以看到 [gensym](http://clojuredocs.org/clojure_core/clojure.core/gensym) 名称：

```clojure
user=> (macroexpand '(m))
(let* [x__681__auto__ 1] x__681__auto__)
```

## #? - 阅读器条件

阅读器条件旨在允许不同的 Clojure 方言共享共同代码。 读者条件的行为类似于传统的 `cond`。 使用的语法是 `#?`，看起来像这样：

```clojure
#?(:clj  (Clojure expression)
   :cljs (ClojureScript expression)
   :cljr (Clojure CLR expression)
   :default (fallthrough expression))
```

## #?@ - 分隔阅读器条件

拼接读取条件的语法是 `#?@`，用于将列表拼接到包含的形式中。因此，Clojure 阅读器会将以下代码：

```clojure
(defn build-list []
  (list #?@(:clj  [5 6 7 8]
            :cljs [1 2 3 4])))
```

读取为：

```clojure
(defn build-list []
  (list 5 6 7 8))
```

`#?@` 读取条件用于在 Clojure 和 ClojureScript 两种平台上选择不同的数据结构，以便编写跨平台代码。即如果代码在 Clojure（JVM）环境下运行，`#?@` 会解包 `[5 6 7 8]` 作为 `list` 的参数。如果是在 ClojureScript 环境下运行，则是 [1 2 3 4]。

## `*var-name*` - "Earmuffs"

“耳罩”（一对星号夹在变量名的两端）是许多 Lisp 中的一种命名约定，用于表示特殊变量。在 Clojure 中，这通常用来表示动态变量，即可以根据动态作用域而变化的变量。耳罩起到警告作用，提示不要假设变量的状态。这只是一个约定，不是强制规则。

Clojure 核心中的示例包括 `*out*` 和 `*in*`，它们分别代表 Clojure 的标准输入和输出流。

## `>!!`, `<!!`, `>!` 和 `<!` - core.async channel macros

这些符号是 `core.async` 中的通道操作符，`core.async` 是一个用于基于通道的异步编程（特别是 [CSP - 通信顺序进程](http://en.wikipedia.org/wiki/Communicating_sequential_processes)）的 Clojure/ClojureScript 库。

为了方便理解，可以将通道类比为一个队列，允许数据的存放和提取，这些符号支持这一简单的 API：

- `>!!` 和 `<!!` 分别是阻塞的放入和取出操作
- `>!` 和 `<!` 是非阻塞的放入和取出操作

两者的区别在于，阻塞版本在 `go` 块外操作，并阻塞它们运行的线程。

```clojure
user=> (def my-channel (chan 10)) ; create a channel
user=> (>!! my-channel "hello")   ; put stuff on the channel
user=> (println (<!! my-channel)) ; take stuff off the channel
hello
```

非阻塞版本必须在 `go` 块中执行，否则会抛出异常。

```clojure
user=> (def c (chan))
#'user/c
user=> (>! c "nope")
AssertionError Assert failed: >! used not in (go ...) block
nil  clojure.core.async/>! (async.clj:123)
```

虽然它们之间的区别超出了本指南的范围，但基本上 `go` 块自行管理资源，暂停代码执行而不会阻塞线程。这使得异步执行的代码看起来像同步执行，减少了代码库中管理异步代码的复杂性。

## `<symbol>?` - 谓词后缀

在符号末尾加上 `?` 是许多支持特殊字符的语言中的常见命名约定，用于表示这是一个谓词，即该函数提出一个问题。例如，想象一个处理缓冲区操作的 API：

```clojure
(def my-buffer (buffers/create-buffer [1 2 3]))
(buffers/empty my-buffer)
```

一眼就可以判断函数 `empty` 是否：

1. 返回 `true` 表示传入的缓冲区为空，或
2. 清空缓冲区

尽管作者可以将 `empty` 重命名为 `is-empty`，但 Clojure 的丰富符号命名方式允许我们更符号化地表达意图。

```clojure
(def my-buffer (buffers/create-buffer [1 2 3]))
(buffers/empty? my-buffer)
false
```

这只是一个推荐的约定，而非强制要求。

## `<symbol>!` - 不安全操作

在 STM 事务中不安全的函数/宏的名称应以感叹号结尾（如 `reset!`）。

你会经常看到这个符号加在用于修改状态的函数名称后面，例如连接数据存储、更新原子（atom）或关闭文件流。

```clojure
user=> (def my-stateful-thing (atom 0))
#'user/my-stateful-thing
user=> (swap! my-stateful-thing inc)
1
user=> @my-stateful-thing
1
```

这只是一个推荐的约定，而非强制要求。

注意，感叹号通常被称为 "bang"。

## `_` - 无用的参数

当你看到下划线字符用作函数参数或出现在 `let` 绑定中，`_` 是一个常见的命名约定，用来表示该参数不会被使用。

以下是一个使用 `add-watch` 函数的示例，该函数可以在原子（atom）值发生变化时添加回调行为。假设我们有一个原子，希望每次它的值发生变化时打印新值：

```clojure
(def value (atom 0))

(add-watch value nil (fn [_ _ _ new-value]
                       (println new-value))

(reset! value 6)
; prints 6
(reset! value 9)
; prints 9
```

`add-watch` 接受四个参数，但在这种情况下，我们只关心最后一个参数——即原子的最新值，因此对其他参数使用 `_`。

## `,` - 空格字符

在 Clojure 中，`,` 被视为空白符，与空格、制表符或换行符完全相同。因此，在字面集合中从不需要逗号，但通常使用逗号来增强可读性：

```clojure
user=>(def m {:a 1, :b 2, :c 3}
{:a 1, :b 2, :c 3}
```

