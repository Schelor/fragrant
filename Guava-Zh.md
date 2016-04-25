#Guava中文翻译
##用户指南
Guava项目包含很多谷歌的核心库，我们在项目中用到了很多这些Java类库，如集合、缓存、基础类型支持、并发库、通用注解、字符串处理、I/O等。Google的码农每天都在使用这些工具，并切已用之于生产服务中。

查阅Javadoc不总是最有效的方式来学习以及使用一个类库。对于Guava，我们尝试提供可读性良好的，令人愉悦的说明来解释很受欢迎，实用性很强功能特性。

这份wiki指南处于不断更新中，部分还待续. 不仅仅囊括的内容有：

- 知暖知热的基础工具
- 爱不释手的集合工具
- 别具一格的缓存工具
- 时髦潮流的函数式编程
- 锦上添花的并发工具
- 曲径通幽的字符串工具
- 删繁就简的I/O工具
- 细致精密的散列构建
- 即感即知的事件总线
- 喜闻乐见的数学工具
- 相忘于江湖的反射工具
- 零零散散的小贴士


###知暖知热的基础工具

####规避`null`
大意粗心的使用`null`可会造成各种各样的bug. 研读Google的基础代码库后，我们发现95%的集合库都不支持存入`null`值，而且它们在面临`null`时，不会逆来顺受，而是掷地有声的快速作出失败响应。而这种策略对开发者无疑是益处极大的。

此外，`null`的意义还是模棱两可的。很难明确一个`null`返回值是代表何意。例如，`Map.get(key)`返回`null`既可以表示在map中的值是`null`, 也可以表示不存在map中。`null`或表示失败，或表示成功，或表示任何含义。因此，使用非`null`的值可使含义表达得更清晰明朗。

即使如此，但正确的使用`null`却能带来内存和时间上的节省。而且在对象数组里，`null`是不可避免的。与类库比之，在应用代码里，它却是狡黠而迷惑的，诡异而难以细查的bug源头。最值一提的是，一个`null`值不具任何意义。

之于这些原因，许多Guava工具在设计时，都能对`null`作出快速失败响应，即直接拒绝处理`null`。然而，也有适用于`null`场景的解决方案。
当你必需要应对`null`时，Guava提供有很多工具来辅助你简易便利的使用`null`。

#####特定场景(Specific Cases)

如果你想把`null`作为`Set`的值或作为`Map`的key -- 那么不要那么做。
通常你在作检索操作时，明确的指定使用`null`场景，效果会更一目了然。
如果你想把`null`作为`Map`的值 -- 那么就省去那个`Entry`吧。
在`Set`中保持单一的非`null`值吧。

我们很容易混淆这种情况：`Key`在`Map`中的值为`null` 与`Key`在`Map`中固已没有值。
此时，分离这些特有的`key`出来会稍显好处，并需要思考与这些`key`关联的`null`在应用中具体是何意。

如果你想在一个稀疏的`List`里使用null值， 那为什么不考虑使用`Map<Integer, E>`呢。这种实际上还可能更高效一点，还能潜在的更准确的满足你得应用需要。

虑及到空对象这种情况，虽然不一定常常用到，但也时有偶为。例如，为一个枚举类型添加一个表示空含义的常量，就像`java.math.RoundingMode`用`UNNECESSARY `来表示不需作舍入值，如果设定为该值，则抛一例异常而出。

如果真的需要`null`值，但又用到了不能处理`null`的集合实现，可尝试使用多种不同的实现，亦如使用`Collections.unmodifiableList(Lists.newArrayList())` 来代替`ImmutableList`。

#####可选之(Optional)
在许多场景中，开发者用`null`来表示某种不存在的含义： 或许是，本应该有值，但却为空。 亦或许是，本应有一个值，但却无其容值之处(not be found)。似如针对key调用`Map.get`方法返回`null`这种情况。

`Optional<T>`可用一个非空值来替换一个可为空的T类型引用。一个`Optional`或包含一个非空的`T`引用(这种场景称之为present，即表示有值)， 或不包含任何值(这种场景称之为absent，表示缺乏值，即不存在值)。我们不会表述为Optional包含null值。
```java
	Optional<Integer> possible = Optional.of(5);
	possible.isPresent(); // returns true
	possible.get(); // returns 5
```
即使`Optional`与其他程序环境中的`option(选项)`，`maybe(或许)`有些类似，但它却不具有直接的相似性。

我们再次列举一些`Optional`的最常用的操作。
#####创建`Optional`
`Optional`的这类方法都是静态的。
| 方法 |	描述 |
| :---------- | :--------|
|`Optional.of(T)`|创建一个包含非null值的Optional对象，如果给定的值是null,则抛出异常|
|`Optional.absent()` | 返回某类型的不存在的Optional对象|
|`Optional.fromNullable(T)`|根据可空的引用，转换成一个Optional对象。如果值为非空，则作为present, 反之作为absent|
#####查询方法
针对特定值的`Optional`, 这类方法都是非静态的。
| 方法 |	描述 |
| :---------- | :--------|
|`boolean isPresent()`|如果当前Optional包含非空实例，则返回`true`|
|`T get()`|返回Optional包含的T类型非空实例，如果为空，则抛出`IllegalStateException`|
|`T or(T)`|返回Optional包含的T类型实例，如果为空，则返回默认值|
|`T orNull()`|返回Optional包含的T类型实例, 如果为空，则返回`null`。对应的方法为`fromNullable`|
|`Set<T> asSet()`|返回包含有Optional值的不可变的单例`Set`对象，如果没有值，则返回空`Set`。|
除了这些方法，`Optional`还提供有很多便利的工具方法，详细请参见Javadoc。

#####重点

#####便捷方法
	