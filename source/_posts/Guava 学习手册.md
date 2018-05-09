---

title: Guava 学习手册
date: 2016-07-21
updated: 2016-07-21
description: Guava 是 Google 的一个开源项目，包含许多 Google 核心的 Java 常用库
tags:
  - Tools
categories:
  - 技术内幕

---


## com.google.common.base
### 字符串处理
Guava 把字符串处理动作分了几大类，每种动作都有对应的工具类实现，我们可以根据需要使用对应的工具类。

#### CharMatcher
`com.google.common.base.CharMatcher` 是 Guava 提供的用于进行字符匹配的工具类，翻开 `CharMatcher` 的源码，我们知道 `CharMatcher` 是一个抽象类，在其内部 Guava 做了大量默认实现，用来更方便的对字符串做匹配，并通过构造者模式对匹配后的字符串进行处理。

**Note：不支持正则表达式。**

#### Joiner
`com.google.common.base.Joiner` 用来拼接字符串，可以避免大量的手动拼接 `appendTo()` 方法，`Joiner` 可以实现 `Iterable<?>`、`Object[]`、`Map<?, ?>` 类型的拼接。但要实现基本类型数组的拼接就无能为力了，这时就需要借助 `com.google.common.base.primitives` 包的基本类型工具类来实现了。

`Joiner` 底层通过 `StringBuilder` 实现，非线程安全。

#### Splitter
`com.google.common.base.Splitter` 用来分割字符串，可以方便的以任意字符分割字符串，并提供转换为 `Map` 的方法 `MapSplitter.withKeyValueSeparator(String separator)`。

**Note：支持正则表达式分割字符串。**

#### Strings
`com.google.common.base.Strings` 的功能较少，Guava 提供的其它几个工具类已基本可以实现字符串处理的相关功能。

#### Charsets
字符串编码一直是我们很头疼的事情，相信我们都写过这样一行代码：

```java
String s = "trang";
byte[] bytes = s.getBytes(Charset.forName("UTF"));
```

这样写有很多缺点，首先我们的大脑得记住常用的字符串编码，不是 `UTF_`，不是 `UTF+`，只能是 `UTF-` 或者 `UTF`，其次错误输入后的后果也很严重，JVM 会抛出 `java.nio.charset.UnsupportedCharsetException` 异常。

`com.google.common.base.Charsets` 给我们提供了一种便利的方式，`Charsets` 类提供了常见的 `Charset` 编码集，给我们的大脑腾出了位置并且避免了异常。

**Note：Java中提供了类似功能的 `java.nio.charset.StandardCharsets`类。**

#### CaseFormat
`com.google.common.base.CaseFormat` 很机智的替我们解决的大小写转换的问题，并且提供了额外的内容。

在处理数据库与 POJO 的映射时，该类有奇效。

| 类型 | 说明 | 示例
| :-- | :-- | :-- |
| LOWER\_CAMEL | 小写驼峰 | lowerCamel
| LOWER\_HYPHEN | 小写连接符 | lower\-hyphen
| LOWER\_UNDERSCORE | 小写下划线 | lower\_underscore
| UPPER\_CAMEL | 大写驼峰 | UpperCamel
| UPPER\_UNDERSCORE | 大写下划线 | UPPER_UNDERSCORE

### <span id="function">函数式编程</span>
在 Java8 面世之前，Guava 一直是函数式编程的不二之选，但**过度使用 Guava 函数式编程会导致冗长、混乱、可读性差而且低效的代码**。这是迄今为止最容易（也是最经常）被滥用的部分，如果你想通过函数式风格达成一行代码，致使这行代码长到荒唐，Guava 团队会泪流满面。

`Predicate` 和 `Function` 是函数式编程中最重要的两个接口，通常通过匿名内部类的方式实现自己的函数，也可以通过对应的工具类使用 Guava 已为你写好的函数。

#### Predicate
`com.google.common.base.Predicate<T>`，断言预期结果，如果与预期不符则放弃。`Predicate` 只声明了一个方法 `boolean apply(T input)` ，使用时只需要实现断言表达式即可。

`CharMatcher` 和 `Range` 也是通过 `Predicate` 实现的。

常见使用 Predicate 的方法：

```java
Iterables.filter(Iterable<T> unfiltered, Predicate<? super T> predicate)
FluentIterable.filter(Predicate<? super T> predicate)
Collections.filter(Collection<E> unfiltered, Predicate<? super E> predicate)
Sets.filter(Set<T>, Predicate<? super T>)
Maps.filterKeys(Map<K, V> unfiltered, Predicate<? super K> keyPredicate)
Maps.filterValues(Map<K, V> unfiltered, Predicate<? super V> valuePredicate)
Maps.filterEntries(Map<K, V> unfiltered, Predicate<? super Entry<K, V>> entryPredicate)
Multimaps.filter(Predicate<? super E> predicate)
```

#### Function
`com.google.common.base.Function<F, T>`，函数，`Function` 最常用的功能是转换集合，同样只需要实现 `T apply(F input)` 即可愉快的玩耍了。

常见使用 Function 的方法：

```
与`Predicate`基本一致
```

#### Supplier
`com.google.common.base.Supplier<T>` 可以对传入的对象进行包装构建后再输出。与前两个函数接口一样，`Supplier` 只提供了一个供实现的方法 `T get()`，用于获取包装后的对象。由于 `Supplier` 在对象的外层，所以 `Supplier` 的一个重要作用是赋予对象懒加载的特性。

### 其它工具类
#### Optional
Guava 用 `com.google.common.base.Optional` 表示可能为`null`的`T`类型引用。一个 `Optional` 实例可能包含非 `null` 的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是 `null` 值，而是用存在或缺失来表示。但 `Optional` 从不会包含 `null` 值引用。

`Optional` 最大的优点在于它是一种傻瓜式的防护。`Optional` 迫使你积极思考引用缺失的情况，因为你必须显式地从 `Optional` 获取引用。

#### Stopwatch
`com.google.common.base.Stopwatch`是一种灵活的代替`System.currentTimeMillis()`和`System.nanoTime()`的方式。

你可以把`Stopwatch`想象成一个秒表，它支持暂停和重置，并且支持`java.util.concurrent.TimeUnit`的任何计时单位。

## com.google.common.collect
### 拓展集合
Guava对Java默认的集合做了大量拓展，以实现不同的业务需求。

#### ImmutabelMap
Guava的`Immutable`系列被很多人推崇，`Immutable`对象的数据在创建时提供，并且在整个生命周期内不可变，这样带来了一些好处：

    线程安全
    节省空间，有效利用内存
    可当做常量使用

以前我们常使用`Collections.unmodifiableXxx()`来定义常量集合，但我们都知道它是有缺陷的。以后当我们使用常量集合时，推荐大家使用Guava的`Immutable集合`，Guava把所有集合类都建立了对应的不可变集合。

**常见ImmutabelMap实现类：**

| 可变集合类型 | 来源 | Guava不可变集合
| :-- | :-- | :-- |
| Collection | JDK | ImmutableCollection
| List | JDK | ImmutableList
| Set | JDK | ImmutableSet
| SortedSet | JDK | ImmutableSortedSet
| Map | JDK | ImmutableMap
| SortedMap | JDK | ImmutableSortedMap
| Multiset | Guava | ImmutableMultiset
| SortedMultiset | Guava | ImmutableSortedMultiset
| Multimap | Guava | ImmutableMultimap
| ListMultimap | Guava | ImmutableListMultimap
| SetMultimap | Guava | ImmutableSetMultimap
| BiMap | Guava | ImmutableBiMap
| ClassToInstanceMap | Guava | ImmutableClassToInstanceMap
| TypeToInstanceMap | Guava | ImmutableTypeToInstanceMap
| Table | Guava | ImmutableTable
| RangeSet | Guava | ImmutableRangeSet
| RangeMap | Guava | ImmutableRangeMap


*Note：*
*当进行`add`、`remove`等操作时抛出`java.lang.UnsupportedOperationException`，该异常为运行时异常，并不会在编译时提醒你，需要开发时注意*
*所有`ImmutableMap`均不支持null值*
*所有`ImmutableMap`均不支持插入相同的key*

#### MultiSet
`com.google.common.collect.Multiset<E>`和`Set`的区别是可以保存多个相同的对象。在JDK中，`List`和`Set`有一个基本的区别，就是List有序且可重复，而Set不能有重复，也不保证顺序（有些实现有顺序，例如`LinkedHashSet`和`SortedSet`等）所以`Multiset`占据了`List`和`Set`之间的一个灰色地带：允许重复，但不保证顺序。

常见使用场景：`Multiset`有一个有用的功能，就是跟踪每种对象的数量，所以你可以用来进行数字统计。
```
HashMultiset.<Integer>create().count(element);
```

**常见MultiSet实现类：**

| Value类型 | 来源 | Gauva Multimap
| :-- | :-- | :-- |
| Enum | JDK | EnumMultiset
| HashMap | JDK | HashSetMultiset
| LinkedHashMap | JDK | LinkedHashMultiset
| TreeMap | JDK | TreeMultiset
| ConcurrentHashMap | JDK | ConcurrentHashMultiset
| ImmutableMap | Guava | ImmutableMultiset

#### MultiMap
Guava中提供了形如`Map<K, List<V>>`或者`Map<K, Set<V>>`的新集合`com.google.common.collect.Multimap<K, V>`，方便的实现了把一个键对应到多个值的数据结构。

**常见MultiMap实现类：**

| Value类型 | 来源 | Gauva Multimap
| :-- | :-- | :-- |
| ArrayList | JDK | ArrayListMultimap
| LinkedList | JDK | LinkedListMultimap
| HashSet | JDK | HashMultimap
| LinkedHashSet | JDK | LinkedHashMultimap
| TreeSet | JDK | TreeMultimap
| ImmutableList | Guava | ImmutableListMultimap
| ImmutableSet | Guava | ImmutableSetMultimap

*Note：*
*`MultiMap`并不是Map*
*除了两个`ImmutableMap`，其它均支持null键和null值*

#### BiMap
`com.google.common.collect.BiMap<K, V>`是一个双向Map，在`BiMap`中，键值都是唯一的。

**常见BiMap实现类：**

| Value类型 | 来源 | Gauva Multimap
| :-- | :-- | :-- |
| HashMap | JDK | HashBiMap
| EnumMap | JDK | EnumBiMap
| EnumMap | JDK | EnumHashBiMap
| ImmutableMap | Guava | ImmutableBiMap

#### Table
`com.google.common.collect.Table<R, C, V>`代替了形如`Map<FirstName, Map<LastName, Person>>`的集合，通过行和列来确定唯一的值。

**常见Table实现类：**

| Value类型 | 来源 | Gauva Multimap
| :-- | :-- | :-- |
| HashMap | JDK | HashBasedTable
| TreeMap | JDK | TreeBasedTable
| ImmutableMap | Guava | ImmutableTable
|      | Guava | ArrayTable

#### ClassToInstanceMap
`com.google.common.collect.ClassToInstanceMap<B>`是一种特殊的Map：它的键是类型，而值是符合键所指类型的对象。

Guava提供了两种有用的`ClassToInstanceMap`实现：`com.google.common.collect.MutableClassToInstanceMap`和 `com.google.common.collect.ImmutableClassToInstanceMap`。

为了扩展Map接口，`ClassToInstanceMap`额外声明了两个方法：`T getInstance(Class<T>)` 和`T putInstance(Class<T>, T)`，从而避免强制类型转换，同时保证了类型安全。

```java
ClassToInstanceMap<Object> map = MutableClassToInstanceMap.create();
map.putInstance(Integer.class, );
map.putInstance(String.class, "");
```

*Note：通常泛型<B>为`java.lang.Object`*

#### Range

```java
@TODO
com.google.common.collect.Range<C extends Comparable>
com.google.common.collect.RangeSet<C extends Comparable>
com.google.common.collect.RangeMap<K extends Comparable, V>
```

### 集合工具类
Guava对JDK内置和Guava拓展的集合均开发了工具类，分别为`Collections`、`Iterables`、`Lists`、`Sets`、`Maps`、`Queues`、`Multisets`、`Multimaps`、`Tables`，里面囊括了异常强大的静态工具方法。

*Guava对拓展的具体集合实现类没有提供基于工具类的初始化方法，而是直接在集合类中提供了静态工厂方法。*

#### Collections
`com.google.common.collect.Collections`提供的方法不多，最常用的方法是函数编程的两个方法，具体内容在 [函数式编程](#function)：

```java
Collection<E> filter(Collection<E> unfiltered, Predicate<? super E> predicate)
Collection<T> transform(Collection<F> fromCollection, Function<? super F, T> function)
```

#### Iterables
`com.google.common.collect.Iterables`为所有实现`java.lang.Iterable<T>`接口的类提供了大量实用方法。如果你使用了`Iterator`，Guava同样为你提供了`Iterators`，它们的作用基本一致。

`Iterables`并不会傻瓜式的任何方法都会遍历对象，而是很精明的通过`instanceof`判断对象实际的类型，如果匹配上则调用该类型的方法，匹配不上才会遍历。

*Note：建议用`Iterables`代替`Collections`*

#### Lists
`com.google.common.collect.Lists`提供了创建`List`的工厂方法，其余有用的有两个：

```java
List<List<B>> cartesianProduct(List<? extends List<? extends B>> lists)
List<T> reverse(List<T> list)
```

`Lists`没有函数编程的`filter()`方法，需要使用工厂方法代替实现：
```
Lists.newArrayList(Iterables.filter(from, Predicates.contains(Pattern.compile("[-]"))));
```

#### Sets
由于`Set`的不重复特性，我们常用`Set`实现一些算法，而`com.google.common.collect.Sets`贴合实际的满足了我们的要求，提供了交集、并集、差集等多种运算方式，并定义了视图`com.google.common.collect.Sets.SetView`来展示结果。

```java
SetView<E> intersection(final Set<E> set, final Set<?> set)
SetView<E> union(final Set<? extends E> set, final Set<? extends E> set)
SetView<E> difference(final Set<E> set, final Set<?> set)
SetView<E> symmetricDifference(final Set<? extends E> set, final Set<? extends E> set)
```

`MultiSet`的工具类为`com.google.common.collect.MultiSet`。

#### Maps
`com.google.common.collect.Maps`提供了`Map`、`SortedMap`、`BiMap`的工厂方法及工具，`Multimap`的工具类为`com.google.common.collect.Multimaps`。

`Maps`中比较常用的方法是`ImmutableMap<K, V> uniqueIndex(Iterable<V> values, Function<? super V, K> keyFunction)`。

### 比较器
#### ComparisonChain
在Java中，我们实现排序往往有两种方式：

    要排序的对象实现`java.lang.Comparable<T>`接口，重写`compareTo(T o)`方法
    定义排序对象，实现`java.util.Comparator<T>`接口，重写`compare(T o, T o)`方法

重写比较方法是件麻烦的事情，Guava又一次帮我们逃离苦海，利用`com.google.common.collect.ComparisonChain`轻松愉快的完成比较方法。
```
public int compare(Cut cut, Cut cut) {
    // 按照Rorate -> X -> Y 排序
    return ComparisonChain.start()
              .compare(cut.getRotate(), cut.getRotate())
              .compare(cut.getX(), cut.getX())
              .compare(cut.getY(), cut.getY())
              .result();
}
```

#### Ordering
`ComparisonChain`带来的功能仍然比较单一，而Guava同时为我们提供了异常强大且方便的链式调用比较器`com.google.common.collect.Ordering<T>`，`Ordering`实现了`Comparator`接口，所以完全可以用`Ordering`替代`Comparator`。

`Ordering`提供了大量的默认实现，每个比较器都提供了常见的链式调用方法，大家可以根据实际情况创建自己的比较器。

*Note：*
*基本类型的比较可以使用`com.google.common.base.primitives`包*
*Java中提供了类似功能的`java.util.Comparators`类*
## com.google.common.cache
缓存是一个成熟的系统中必不可少的一环，合理利用缓存可以显著提升系统响应速度，减少I/O压力，Java常见的缓存有`Redis`、`Memcached`、`EhCache`等，而今天我们介绍的是Guava提供的本地缓存`Guava Cache`。

`Guava Cache`在很多场景下都是相当有用的，比如初始化查找树，我们只需要对不同的树初始化一次，以后直接调用即可。

`Guava Cache`与`ConcurrentMap`很相似，但不完全一样。最基本的区别是`ConcurrentMap`会一直保存所有添加的元素，直到显式地移除。相对地，`Guava Cache`为了限制内存占用，通常都设定为自动回收元素。

### 应用缓存
#### Cache
`com.google.common.cache.Cache<K, V>`是`Guava Cache`的基本接口，Guava为我们提供了一个默认实现`LocalManualCache`，我们可以通过`CacheBuilder`工具类的工厂方法来创建`LocalManualCache`对象。

```
CacheBuilder.newBuilder().build()
```

#### CacheLoader
在使用`Guava Cache`前，首先问自己一个问题：有没有合理的默认方法来加载与键关联的值？如果没有，`Cache`就是为你打造的，你需要在获取缓存的值时传入一个`Callable`实例来保证`Guava Cache`的作用。如果有，那么`CacheLoader`更适合你，而这也是今天的重点。

`com.google.common.cache.LoadingCache<K, V>`是实现了`CacheLoader`的`Guava Cache`，与创建`Cache`实例时仅有一点不同，只需要调用`build()`的重载方法即可。

```
CacheBuilder.newBuilder().build(cacheLoader);
```

正因为`LoadingCache`有了默认的加载方法，所以只需要调用`get(K key)`即可得到值。

### 自定义缓存
#### CacheBuilder
从之前的代码中大家都看到了，`Guava Cache`的实例正是通过`CacheBuilder`创建的，事实上，`CacheBuilder`的作用远远不止这些，掌握好`CacheBuilder`，享受自定义`Guava Cache`的乐趣吧。

**自定义CacheBuilder参数：**

| 方法 | 参数 | 说明
| :-- | :-- | :-- |
| initialCapacity | int | 初始化容量
| maximumSize | long | 最大容量
| weigher | Weigher | 权重函数
| maximumWeight | long | 最大权重
| concurrencyLevel | int | 并发级别
| expireAfterAccess | long, TimeUnit | 上次访问给定时间后回收
| expireAfterWrite | long, TimeUnit | 缓存写入给定时间后回收
| refreshAfterWrite | long, TimeUnit | 缓存写入给定时间后更新
| removalListener | RemovalListener | 移除监听器(同步)
| recordStats |  | 统计状态

`CacheBuilder`提供了一系列的参数供我们个性化，主要是为了缓存的回收。`CacheBuilder`创建的实例并不会自动清理失效的缓存，而是在你进行读或写操作的时候顺带维护。这样做的原因在于，如果要自动地持续清理缓存，就必须有一个线程，这个线程会和用户操作竞争共享锁。


## com.google.common.net
**常用类**
**@Beta:** `HostAndPort` `HostSpecifier` `InetAddresses` `InternetDomainName` `MediaType` `PercentEscaper` `UrlEscapers`

**@Release:** `HttpHeaders`
Guava中的`com.google.common.net`包目前提供的功能较少，而且大多类都标注了@Beta的注解，在Guava中标记@Beta表示这个类还不稳定，有可能在以后的版本中变化，或者去掉，所以不建议大量使用，这里也是只做简单的介绍。

### HttpHeaders
先介绍下唯一一个没有@Beta注解的类`HttpHeaders`，这个类中并没有实质的方法，只是定义了一些Http头名称的常量，通常如果需要我们会自己定义这些常量，如果你引用了Guava包，那么就不再建议我们自己定义这些头名称的常量了，直接用它定义的即可。

这里面应该有几乎所有的Http头名称，例如：`X_FORWARDED_FOR`，`CONTENT_TYPE`，`ACCEPT`等，用法也没有必要介绍了，直接引用常量就可以了。

### HostAndPort
有时候我们需要得到请求的ip，这时候通畅需要自己写方法解析url。而Guava给我们提供的`HostAndPort`类正是做这种事情的利器，可以从字符串中得到ip和port
```
HostAndPort.fromString(String hostPortString)
```

## 参考资料：
1. [Guava API](http://google.github.io/guava/releases/snapshot/api/docs/)
2. [瓜娃系列](http://ajoo.iteye.com/category/119082)
3. [Guava官方文档](http://ifeve.com/google-guava/)
4. [Guava教程](http://www.yiibai.com/guava/)

[1]: http://docs.oracle.com/javase/1.5.0/docs/guide/collections/designfaq.html
