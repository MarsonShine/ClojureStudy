# Spec 指南

## 快速开始

### spec库

[spec](https://clojure.org/about/spec) 库（[API 文档](https://clojure.github.io/spec.alpha)）用于指定数据结构、验证或符合数据，并可以基于spec生成数据。

要使用 `spec`，需要声明对Clojure 1.9.0或更高版本的依赖：

```clojure
[org.clojure/clojure "1.12.0"]
```

要开始使用 `spec`，可以在 REPL 中引入 `clojure.spec.alpha` 命名空间：

```clojure
(require '[clojure.spec.alpha :as s])
```

或者在你的命名空间中包含 `spec`：

```clojure
(ns my.ns
  (:require [clojure.spec.alpha :as s]))
```

## 断言（Predicates）

每个 `spec` 描述了一组允许的值。构建 `spec` 有几种方式，可以组合它们来构建更复杂的 `spec`。

任何接受单个参数并返回真值的 Clojure 函数都是一个有效的断言 `spec`。我们可以使用 `conform` 函数检查特定数据值是否符合 `spec`：

```clojure
(s/conform even? 1000)
;;=> 1000
```

`conform` 函数接受一个可以是 `spec` 的参数和一个数据值。在这里，我们传入了一个断言，它被隐式转换为 `spec`。返回值是“符合的”值。在此例中，符合值与原值相同——稍后我们会看到不同的情况。如果值不符合 `spec`，则返回特殊值 `:clojure.spec.alpha/invalid`。

如果你不需要符合值或检查 `:clojure.spec.alpha/invalid`，可以使用辅助函数 `valid?` 来返回一个布尔值。

```clojure
(s/valid? even? 10)
;;=> true
```

注意，`valid?` 同样会隐式地将断言函数转换为 `spec`。`spec` 库允许利用已有的所有函数——不需要特别的断言字典。更多示例：

```clojure
(s/valid? nil? nil)  ;; true
(s/valid? string? "abc")  ;; true

(s/valid? #(> % 5) 10) ;; true
(s/valid? #(> % 5) 0) ;; false

(import java.util.Date)
(s/valid? inst? (Date.))  ;; true
```

集合还可用作匹配一个或多个字面值的谓词：

```clojure
(s/valid? #{:club :diamond :heart :spade} :club) ;; true
(s/valid? #{:club :diamond :heart :spade} 42) ;; false

(s/valid? #{42} 42) ;; true
```

## 注册表

到目前为止，我们一直在直接使用 `specs`。然而，`spec` 提供了一个中央注册表，用于全局声明可重用的 `specs`。注册表将一个带命名空间的关键字与 `spec` 关联。使用命名空间可确保我们在不同库或应用中定义的 `specs` 不会发生冲突。

可以使用 `s/def` 来注册 `spec`。可以在一个有意义的命名空间中注册 `spec`（通常是你控制的命名空间）。

```clojure
(s/def :order/date inst?)
(s/def :deck/suit #{:club :diamond :heart :spade})
```

已注册的 `spec` 标识符可以在前面讨论的操作（如 `conform` 和 `valid?`）中替代 `spec` 定义。

```clojure
(s/valid? :order/date (Date.))
;;=> true
(s/conform :deck/suit :club)
;;=> :club
```

后续会看到，注册的 `specs` 可以（也应当）在组合 `specs` 时使用。

> ### `Spec` 名称
>
> `Spec` 名称始终是带完全限定的关键字。通常情况下，Clojure代码应使用足够独特的关键字命名空间，以避免与其他库提供的 `specs` 冲突。如果你在编写一个公共库，`spec` 命名空间应包含项目名称、网址或组织名称。对于私有组织内部的代码，可以使用较短的名称——重要的是要确保其足够独特以避免冲突。
>
> 在本指南中，为了简单起见，通常会使用较短的限定名称。
>

一旦将 `spec` 添加到注册表中，可以通过 `doc` 命令找到并打印它：

```clojure
(doc :order/date)
-------------------------
:order/date
Spec
  inst?

(doc :deck/suit)
-------------------------
:deck/suit
Spec
  #{:spade :heart :diamond :club}
```

## 组合断言

最简单的 `spec` 组合方法是使用 `and` 和 `or`。我们可以使用 `s/and` 创建一个组合了多个断言的 `spec`：

```clojure
(s/def :num/big-even (s/and int? even? #(> % 1000)))
(s/valid? :num/big-even :foo) ;; false
(s/valid? :num/big-even 10) ;; false
(s/valid? :num/big-even 100000) ;; true
```

也可以使用 `s/or` 来指定两种或多种备选情况：

```clojure
(s/def :domain/name-or-id (s/or :name string?
                                :id   int?))
(s/valid? :domain/name-or-id "abc") ;; true
(s/valid? :domain/name-or-id 100) ;; true
(s/valid? :domain/name-or-id :foo) ;; false
```

这个 `or` `spec` 是我们遇到的第一个涉及有效性检查过程中选择的例子。每个分支都有一个标签（这里是 `:name` 和 `:id` 之间），这些标签为分支命名，用于理解或丰富从 `conform` 和其他 `spec` 函数返回的数据。

当一个 `or` 被符合（`conform`）时，它会返回一个带有标签名称和符合值的向量：

```clojure
(s/conform :domain/name-or-id "abc")
;;=> [:name "abc"]
(s/conform :domain/name-or-id 100)
;;=> [:id 100]
```

许多检查实例类型的断言不允许 `nil` 作为有效值（如 `string?`、`number?`、`keyword?` 等）。可以使用 [nilable](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/nilable) 函数将 `nil` 作为有效值：

```clojure
(s/valid? string? nil)
;;=> false
(s/valid? (s/nilable string?) nil)
;;=> true
```

## `explain`

[explain](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/explain) 是 `spec` 中的另一个高级操作，用于报告（输出到 `*out*`）值不符合 `spec` 的原因。我们来看一下 `explain` 对于一些不符合示例的输出：

```clojure
(s/explain :deck/suit 42)
;; 42 - failed: #{:spade :heart :diamond :club} spec: :deck/suit

(s/explain :num/big-even 5)
;; 5 - failed: even? spec: :num/big-even

(s/explain :domain/name-or-id :foo)
;; :foo - failed: string? at: [:name] spec: :domain/name-or-id
;; :foo - failed: int? at: [:id] spec: :domain/name-or-id
```

让我们仔细观察最后一个例子的输出。首先注意到，这里报告了两个错误——`spec` 会评估所有可能的备选路径，并在每条路径上报告错误。每个错误的组成部分包括：

- `val` - 不匹配的用户输入中的值
- `spec` - 正在评估的 `spec`
- `at` - 路径（关键字的向量），指示发生错误的 `spec` 中的位置。路径中的标签对应于 `spec` 中任何带标签的部分（`or` 或 `alt` 中的备选部分、`cat` 中的部分、映射中的键等）
- `predicate` - 未满足的实际断言
- `in` - 嵌套数据中通向失败值的键路径。在这个例子中，顶层值是失败的值，所以这是一个空路径，省略了。

从第一个错误中可以看到，值 `:foo` 在 `spec` `:domain/name-or-id` 的 `:name` 路径上未满足 `string?` 断言。第二个错误类似，但失败在 `:id` 路径上。实际值是关键字，因此都不匹配。

除了 `explain`，还可以使用 `explain-str` 接收错误消息作为字符串，或 `explain-data` 以数据形式接收错误。

```clojure
(s/explain-data :domain/name-or-id :foo)
;;=> #:clojure.spec.alpha{
;;     :problems ({:path [:name],
;;                 :pred clojure.core/string?,
;;                 :val :foo,
;;                 :via [:domain/name-or-id],
;;                 :in []}
;;                {:path [:id],
;;                 :pred clojure.core/int?,
;;                 :val :foo,
;;                 :via [:domain/name-or-id],
;;                 :in []})}
```

> 此结果也演示了 Clojure 1.9中添加的命名空间映射字面量语法。映射可以用 `#:` 或 `#::`（自动解析）前缀，以指定映射中所有键的默认命名空间。在此示例中，这等价于 `{:clojure.spec.alpha/problems …}`。

## 实体映射（Entity Maps）

Clojure 程序广泛使用映射来传递数据。在其他库中，常见的方法是描述每个实体类型，将它包含的键和这些键的值结构结合在一起。而 `spec` 则通过在属性级别（键 + 值）为各个属性赋予含义，然后使用集合语义（基于键）将它们收集到映射中。这种方法使我们可以在属性级别开始为不同库和应用程序分配和共享语义。

例如，大多数 `Ring` 中间件函数使用非限定键来修改请求或响应映射。然而，每个中间件也可以使用带有注册语义的命名空间键，这些键可以被检查以确保符合规范，从而创建更具协作性和一致性的系统。

在 `spec` 中，实体映射可以使用 `keys` 定义：

```clojure
(def email-regex #"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,63}$")
(s/def :acct/email-type (s/and string? #(re-matches email-regex %)))

(s/def :acct/acctid int?)
(s/def :acct/first-name string?)
(s/def :acct/last-name string?)
(s/def :acct/email :acct/email-type)

(s/def :acct/person (s/keys :req [:acct/first-name :acct/last-name :acct/email]
                            :opt [:acct/phone]))
```

此代码注册了一个 `:acct/person` 的 `spec`，其必需键包括 `:acct/first-name`、`:acct/last-name` 和 `:acct/email`，可选键为 `:acct/phone`。映射 `spec` 不会为属性指定值的 `spec`，只会指定哪些属性是必需的或可选的。

当检查映射是否符合规范时，它会执行两个操作：检查是否包含必需的属性，并检查每个已注册的键是否符合规范。后续会看到可选属性的用途。此外，所有属性均通过 `keys` 检查，而不仅仅是 `:req` 和 `:opt` 中列出的键。因此，裸 `s/keys` 是有效的，可以检查映射的所有属性，而不检查哪些键是必需或可选的。

```clojure
(s/valid? :acct/person
  {:acct/first-name "Bugs"
   :acct/last-name "Bunny"
   :acct/email "bugs@example.com"})
;;=> true
```

缺少必需键检查失败：

```clojure
(s/explain :acct/person
  {:acct/first-name "Bugs"})
;; #:acct{:first-name "Bugs"} - failed: (contains? % :acct/last-name) spec: :acct/person
;; #:acct{:first-name "Bugs"} - failed: (contains? % :acct/email) spec: :acct/person
```

属性不符合规范检查失败：

```clojure
(s/explain :acct/person
  {:acct/first-name "Bugs"
   :acct/last-name "Bunny"
   :acct/email "n/a"})
;; "n/a" - failed: (re-matches email-regex %) in: [:acct/email]
;;   at: [:acct/email] spec: :acct/email-type
```

我们可以通过 `explain` 错误输出分析最后一个示例：

- `in` - 数据中导致失败的值路径（在此为person实例中的键）
- `val` - 失败的值，这里是 `"n/a"`
- `spec` - 失败的 `spec`，这里为 `:acct/email-type`
- `at` - `spec` 中失败值的位置路径
- `predicate` - 失败的断言，这里是 `(re-matches email-regex %)`

许多现有的 Clojure 代码并不使用带有命名空间键的映射，因此键也可以通过 `:req-un` 和 `:opt-un` 指定无命名空间的必需和可选键。这些变体指定用于查找其规范的命名空间键，但映射仅检查无命名空间版本的键。

例如使用非限定键的 `person` 映射，但仍然根据前面注册的命名空间 `specs` 检查符合性：

```clojure
(s/def :unq/person
  (s/keys :req-un [:acct/first-name :acct/last-name :acct/email]
          :opt-un [:acct/phone]))

(s/conform :unq/person
  {:first-name "Bugs"
   :last-name "Bunny"
   :email "bugs@example.com"})
;;=> {:first-name "Bugs", :last-name "Bunny", :email "bugs@example.com"}

(s/explain :unq/person
  {:first-name "Bugs"
   :last-name "Bunny"
   :email "n/a"})
;; "n/a" - failed: (re-matches email-regex %) in: [:email] at: [:email]
;;   spec: :acct/email-type

(s/explain :unq/person
  {:first-name "Bugs"})
;; {:first-name "Bugs"} - failed: (contains? % :last-name) spec: :unq/person
;; {:first-name "Bugs"} - failed: (contains? % :email) spec: :unq/person
```

也可以使用非限定键验证记录属性：

```clojure
(defrecord Person [first-name last-name email phone])

(s/explain :unq/person
           (->Person "Bugs" nil nil nil))
;; nil - failed: string? in: [:last-name] at: [:last-name] spec: :acct/last-name
;; nil - failed: string? in: [:email] at: [:email] spec: :acct/email-type

(s/conform :unq/person
  (->Person "Bugs" "Bunny" "bugs@example.com" nil))
;;=> #user.Person{:first-name "Bugs", :last-name "Bunny",
;;=>              :email "bugs@example.com", :phone nil}
```

Clojure 中经常使用“关键字参数”，其中关键字键和值以序列化数据结构作为选项传递。`spec` 提供了专门的 [keys*](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/keys*) 来支持此模式。`keys*` 的语法和语义与 `keys` 相同，但可以嵌入到序列化的正则表达式结构中。

```clojure
(s/def :my.config/port number?)
(s/def :my.config/host string?)
(s/def :my.config/id keyword?)
(s/def :my.config/server (s/keys* :req [:my.config/id :my.config/host]
                                  :opt [:my.config/port]))
(s/conform :my.config/server [:my.config/id :s1
                              :my.config/host "example.com"
                              :my.config/port 5555])
;;=> #:my.config{:id :s1, :host "example.com", :port 5555}
```

有时，将实体映射分部分声明会很方便，原因可能是实体映射的需求来自不同的来源，或者存在一组通用键和一些特定于变体的部分。可以使用 `s/merge` 来将多个 `s/keys` 规范合并为一个，组合它们的需求。例如，可以定义两个 `keys` 规范来表示通用的动物属性和一些狗特有的属性。狗实体本身可以描述为这两个属性集合的合并：

```clojure
(s/def :animal/kind string?)
(s/def :animal/says string?)
(s/def :animal/common (s/keys :req [:animal/kind :animal/says]))
(s/def :dog/tail? boolean?)
(s/def :dog/breed string?)
(s/def :animal/dog (s/merge :animal/common
                            (s/keys :req [:dog/tail? :dog/breed])))

(s/valid? :animal/dog
  {:animal/kind "dog"
   :animal/says "woof"
   :dog/tail? true
   :dog/breed "retriever"})
;;=> true
```

## 多重规范 (`multi-spec`)

在 Clojure 中，使用映射作为带标签的实体是一种常见情况，这种映射包含一个特殊字段，用于指示映射的“类型”。该类型可能是一个开放的类型集合，通常在类型之间共享一些属性。

如前所述，通过带命名空间的关键字存储在注册表中的属性可以很好地定义所有类型的属性。共享的属性在不同实体类型间自动获得共享语义。然而，我们还希望能够为每种实体类型指定必需的键。为此，`spec` 提供了 `multi-spec`，它利用多方法（`multimethod`）来根据类型标签为开放的实体类型集提供规范。

例如，假设一个 API 接收事件对象，这些对象共享一些通用字段，但也具有特定类型的结构。首先，我们会注册事件的属性：

```clojure
(s/def :event/type keyword?)
(s/def :event/timestamp int?)
(s/def :search/url string?)
(s/def :error/message string?)
(s/def :error/code int?)
```

然后，我们需要一个多方法来定义调度函数，用于选择选择器（这里是 `:event/type` 字段），并根据其值返回适当的 `spec`：

```clojure
(defmulti event-type :event/type)
(defmethod event-type :event/search [_]
  (s/keys :req [:event/type :event/timestamp :search/url]))
(defmethod event-type :event/error [_]
  (s/keys :req [:event/type :event/timestamp :error/message :error/code]))
```

这些方法应忽略其参数，并返回指定类型的 `spec`。在这里，我们已完全定义了两种可能的事件——“搜索”事件和“错误”事件。

最后，我们可以声明我们的 `multi-spec` 并进行测试：

```clojure
(s/def :event/event (s/multi-spec event-type :event/type))

(s/valid? :event/event
  {:event/type :event/search
   :event/timestamp 1463970123000
   :search/url "https://clojure.org"})
;=> true
(s/valid? :event/event
  {:event/type :event/error
   :event/timestamp 1463970123000
   :error/message "Invalid host"
   :error/code 500})
;=> true
(s/explain :event/event
  {:event/type :event/restart})
;; #:event{:type :event/restart} - failed: no method at: [:event/restart]
;;   spec: :event/event
(s/explain :event/event
  {:event/type :event/search
   :search/url 200})
;; 200 - failed: string? in: [:search/url]
;;   at: [:event/search :search/url] spec: :search/url
;; {:event/type :event/search, :search/url 200} - failed: (contains? % :event/timestamp)
;;   at: [:event/search] spec: :event/event
```

让我们花点时间分析最后一个示例中的 `explain` 错误输出。检测到两种不同类型的错误。第一个错误是由于事件中缺少必需的 `:event/timestamp` 键导致的。第二个错误来自无效的 `:search/url` 值（传入了一个数字而非字符串）。我们可以看到与之前 `explain` 错误相同的部分：

- `in` - 数据中导致失败的值的路径。在第一个错误中，由于位于根值，`in` 被省略；在第二个错误中是映射中的键。
- `val` - 失败的值，可以是整个映射，也可以是映射中的单个键。
- `spec` - 实际失败的 `spec`。
- `at` - `spec` 中发生失败值的位置路径。
- `predicate` - 实际失败的断言。

`multi-spec` 方法使我们可以创建一个开放的 `spec` 验证系统，就像多方法（`multimethods`）和协议（`protocols`）一样。新的事件类型可以在之后通过扩展 `event-type` 多方法轻松添加。

## 集合

为其他特殊的集合情况提供了一些辅助函数——[coll-of](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/coll-of)、[tuple](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/tuple) 和 [map-of](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/map-of)。

对于任意大小的同类集合的特殊情况，可以使用 `coll-of` 来指定满足某个断言的元素集合。

```clojure
(s/conform (s/coll-of keyword?) [:a :b :c])
;;=> [:a :b :c]
(s/conform (s/coll-of number?) #{5 10 2})
;;=> #{2 5 10}
```

此外，`coll-of` 可以传入一些关键字参数选项：

- `:kind` - 指定传入集合必须满足的断言，例如 `vector?`。
- `:count` - 指定确切的期望数量。
- `:min-count` 和 `:max-count` - 检查集合的元素数量是否满足 `(<= min-count count max-count)`。
- `:distinct` - 检查所有元素是否唯一。
- `:into` - 指定输出的符合值的集合类型，可为 `[]`、`()`、`{}` 或 `#{}`。如果未指定 `:into`，则使用输入集合的类型。

以下是一个示例，利用这些选项来定义一个包含三个唯一数字的向量，并将其符合值转为集合，同时展示不同无效值类型的一些错误输出：

```clojure
(s/def :ex/vnum3 (s/coll-of number? :kind vector? :count 3 :distinct true :into #{}))
;; s/coll-of 这个函数可以传递多个参数，number? 表示集合都是数字，:kind vector? 表示集合的类型都是向量，元素数量为3，:distinct true向量元素不能重复，上述通过验证就是转成集合形式 :into #{}
(s/conform :ex/vnum3 [1 2 3])
;;=> #{1 2 3}
(s/explain :ex/vnum3 #{1 2 3})   ;; not a vector
;; #{1 3 2} - failed: vector? spec: :ex/vnum3
(s/explain :ex/vnum3 [1 1 1])    ;; not distinct
;; [1 1 1] - failed: distinct? spec: :ex/vnum3
(s/explain :ex/vnum3 [1 2 :a])   ;; not a number
;; :a - failed: number? in: [2] spec: :ex/vnum3
```

> `coll-of` 和 `map-of` 都会对其所有元素进行符合转换，这可能使其不适合大型集合。在这种情况下，可以考虑 [every](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/every) 或针对映射的 [every-kv](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/every-kv)。

虽然 `coll-of` 适用于任意大小的同类集合，但另一种情况是固定大小、具有不同位置字段的集合，且每个位置的字段类型已知。对此可以使用 `tuple`。

```clojure
(s/def :geom/point (s/tuple double? double? double?))
(s/conform :geom/point [1.5 2.5 -0.5])
=> [1.5 2.5 -0.5]
```

注意，对于包含 `x`、`y` 和 `z` 值的“点”结构，我们实际上可以选择三种可能的 `spec`：

1. 正则表达式 - `(s/cat :x double? :y double? :z double?)`
   - 允许匹配嵌套结构（此处不需要）
   - 符合后转为带命名键的映射，基于 `cat` 标签

2. 集合 - `(s/coll-of double?)`
   - 适用于任意大小的同类集合
   - 符合后转为值的向量

3. 元组 - `(s/tuple double? double? double?)`
   - 设计用于固定大小，且位置上字段已知
   - 符合后转为值的向量

在本例中，`coll-of` 也会匹配其他（无效的）值（如 `[1.0]` 或 `[1.0 2.0 3.0 4.0]`），因此它不是合适的选择——我们需要固定字段。选择正则表达式还是 `tuple` 在一定程度上取决于个人偏好，可能还取决于是否期望标签化的返回值或错误输出在两者中表现更佳。

除了通过 `keys` 提供的信息映射支持，`spec` 还提供 `map-of` 用于具有同类键和值断言的映射。

```clojure
(s/def :game/scores (s/map-of string? int?))
(s/conform :game/scores {"Sally" 1000, "Joe" 500})
;=> {"Sally" 1000, "Joe" 500}
```

默认情况下，`map-of` 会验证但不会符合转换键，因为符合转换后的键可能会导致键重复，从而覆盖映射中的条目。如果需要符合转换键，可以传递选项 `:conform-keys true`。

此外，还可以在 `map-of` 上使用与 `coll-of` 相同的各种计数相关选项。

## 序列

有时，会使用顺序数据来编码附加结构（通常是新的语法，常用于宏中）。`spec` 提供了标准的[正则表达式](https://en.wikipedia.org/wiki/Regular_expression)操作符，用于描述顺序数据值的结构：

- [cat](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/cat) - 连接多个断言/模式
- [alt](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/alt) - 选择多个断言/模式之一
- [*](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/*) - 0次或多次匹配断言/模式
- [+](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/%2B) - 1次或多次匹配断言/模式
- [?](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/%3F) - 0次或1次匹配断言/模式

与 `or` 类似，`cat` 和 `alt` 都会为它们的“部分”添加标签，这些标签会在符合后的值中用于标识匹配内容、报告错误等。

例如，用一个包含数量（数字）和单位（关键字）的向量来表示配料。这个数据的 `spec` 使用 `cat` 来指定正确顺序的组件。和断言一样，正则操作符在传递给 `conform`、`valid?` 等函数时会被隐式转换为 `spec`。

```clojure
(s/def :cook/ingredient (s/cat :quantity number? :unit keyword?))
(s/conform :cook/ingredient [2 :teaspoon])
;;=> {:quantity 2, :unit :teaspoon}
```

数据符合后会变为带标签的键的映射。可以使用 `explain` 来检查不符合的数据。

```clojure
;; pass string for unit instead of keyword
(s/explain :cook/ingredient [11 "peaches"])
;; "peaches" - failed: keyword? in: [1] at: [:unit] spec: :cook/ingredient

;; leave out the unit
(s/explain :cook/ingredient [2])
;; () - failed: Insufficient input at: [:unit] spec: :cook/ingredient
```

接下来，我们来看 `*`、`+` 和 `?` 等不同的出现次数操作符：

```clojure
(s/def :ex/seq-of-keywords (s/* keyword?))
(s/conform :ex/seq-of-keywords [:a :b :c])
;;=> [:a :b :c]
(s/explain :ex/seq-of-keywords [10 20])
;; 10 - failed: keyword? in: [0] spec: :ex/seq-of-keywords

(s/def :ex/odds-then-maybe-even (s/cat :odds (s/+ odd?)
                                       :even (s/? even?)))
(s/conform :ex/odds-then-maybe-even [1 3 5 100])
;;=> {:odds [1 3 5], :even 100}
(s/conform :ex/odds-then-maybe-even [1])
;;=> {:odds [1]}
(s/explain :ex/odds-then-maybe-even [100])
;; 100 - failed: odd? in: [0] at: [:odds] spec: :ex/odds-then-maybe-even

;; opts are alternating keywords and booleans
(s/def :ex/opts (s/* (s/cat :opt keyword? :val boolean?)))
(s/conform :ex/opts [:silent? false :verbose true])
;;=> [{:opt :silent?, :val false} {:opt :verbose, :val true}]
```

最后，可以使用 `alt` 在顺序数据中指定备选项。与 `cat` 一样，`alt` 需要为每个备选项打标签，但符合的数据会以标签和值的向量形式表示。

```clojure
(s/def :ex/config (s/*
                    (s/cat :prop string?
                           :val  (s/alt :s string? :b boolean?))))
(s/conform :ex/config ["-server" "foo" "-verbose" true "-user" "joe"])
;;=> [{:prop "-server", :val [:s "foo"]}
;;    {:prop "-verbose", :val [:b true]}
;;    {:prop "-user", :val [:s "joe"]}]
```

> **`s/*`**：表示一个可重复的模式，类似于正则表达式中的 `*`，即匹配零个或多个元素。在这里，它定义了 `:ex/config` 的结构是一个由多个匹配模式组成的序列。
>
> **`s/cat`**：用于定义一个**按顺序匹配的序列**。在这里，`s/cat` 表示一对元素的组合，依次包含“属性”和“值”两个部分。
>
> - **`:prop`**：第一个元素的标签，用于标识“属性”部分。
> - **`string?`**：这是对 `:prop` 的验证条件，要求 `:prop` 必须是一个字符串。
> - **`:val`**：第二个元素的标签，用于标识“值”部分。
> - **`s/alt`**：用于定义 `:val` 可以是多种类型中的一种。`s/alt` 会根据标签选择对应的验证规则。
>   - **`:s`**：如果 `:val` 是字符串，则使用标签 `:s`。
>   - **`:b`**：如果 `:val` 是布尔值，则使用标签 `:b`。
>
> **`string?`** 和 **`boolean?`**：这两个谓词分别检查值是否是字符串和布尔类型。

如果需要查看规范的描述，可以使用 `describe`。我们可以在之前定义的一些 `spec` 上尝试：

```clojure
(s/describe :ex/seq-of-keywords)
;;=> (* keyword?)
(s/describe :ex/odds-then-maybe-even)
;;=> (cat :odds (+ odd?) :even (? even?))
(s/describe :ex/opts)
;;=> (* (cat :opt keyword? :val boolean?))
```

`spec` 还定义了一个额外的正则操作符 `&`，它接受一个正则操作符，并用一个或多个附加断言来约束它。这可以用于创建带有额外约束的正则表达式，否则可能需要自定义断言。例如，想要匹配仅包含偶数个字符串的序列：

```clojure
(s/def :ex/even-strings (s/& (s/* string?) #(even? (count %))))
(s/valid? :ex/even-strings ["a"])  ;; false
(s/valid? :ex/even-strings ["a" "b"])  ;; true
(s/valid? :ex/even-strings ["a" "b" "c"])  ;; false
(s/valid? :ex/even-strings ["a" "b" "c" "d"])  ;; true
```

当组合正则操作符时，它们描述的是单一序列。如果需要为嵌套的顺序集合指定 `spec`，则必须显式调用 [spec](https://clojure.github.io/spec.alpha/clojure.spec.alpha-api.html#clojure.spec.alpha/spec) 来启动新的嵌套正则上下文。例如，描述一个类似 `[:names ["a" "b"] :nums [1 2 3]]` 的序列，需要嵌套的正则表达式来描述内部的顺序数据：

```clojure
(s/def :ex/nested
  (s/cat :names-kw #{:names}
         :names (s/spec (s/* string?))
         :nums-kw #{:nums}
         :nums (s/spec (s/* number?))))
(s/conform :ex/nested [:names ["a" "b"] :nums [1 2 3]])
;;=> {:names-kw :names, :names ["a" "b"], :nums-kw :nums, :nums [1 2 3]}
```

如果去掉 `spec` 的嵌套，这个 `spec` 将匹配一个类似 `[:names "a" "b" :nums 1 2 3]` 的序列。

```clojure
(s/def :ex/unnested
  (s/cat :names-kw #{:names}
         :names (s/* string?)
         :nums-kw #{:nums}
         :nums (s/* number?)))
(s/conform :ex/unnested [:names "a" "b" :nums 1 2 3])
;;=> {:names-kw :names, :names ["a" "b"], :nums-kw :nums, :nums [1 2 3]}
```

