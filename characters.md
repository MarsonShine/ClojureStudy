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