---
    title: Collection
    creator: cjq
    create_time: 2026/03/03
    tags:
      - scala
    categories:
      - [language, scala, 用法]



---



[toc]

# 待整理



### 什么版本的scala sdk collection中包含breakOut

`breakOut` 是 Scala 2.8 标准库 `scala.collection` 包中的一个方法[-3](https://stackoverflow.com/posts/1715681/revisions)，在 Scala 2.13 中被移除--。

下面整理了 `breakOut` 的版本支持情况，并提供了从 2.13 开始的替代方案。

------

#### ⏳ `breakOut` 的版本变迁

| Scala 版本      | `breakOut` 支持状态 | 备注                                                         |
| :-------------- | :------------------ | :----------------------------------------------------------- |
| **2.8 至 2.12** | ✅ **可用**          | 核心功能，需要 `import scala.collection.breakOut`[-3](https://stackoverflow.com/posts/1715681/revisions) |
| **2.13**        | ❌ **已移除**        | 官方推荐使用 `.view` + `.to(Collection)` 组合替代-[-14](https://stackoverflow.com/questions/68723881/conversion-of-breakout-use-iterator-or-view/68725937#68725937) |
| **3.x**         | ❌ **已移除**        | 无此方法，需使用替代方案                                     |

------

#### 💡 `breakOut` 是什么？

`breakOut` 是一个基于 `CanBuildFrom` 机制的工具，作用是在集合类型转换时**避免创建不必要的中间集合**，从而提升性能并简化代码。

例如，将 `List[String]` 直接转为 `Map[Int, String]`，原本需要生成中间 `List[(Int, String)]` 再调用 `.toMap`。使用 `breakOut` 可以一步完成：

scala

复制下载

```
import scala.collection.breakOut
val m: Map[Int, String] = List("London", "Paris").map(x => (x.length, x))(breakOut)
// m: Map(6 -> London, 5 -> Paris)
```



在 2.8 至 2.12 版本中，这种方式有效避免了中间集合的开销，并让集合类型推断更为灵活[-3](https://stackoverflow.com/posts/1715681/revisions)。

------

#### 🔁 从 2.13 开始的替代方案

从 2.13 起，`CanBuildFrom` 被 `BuildFrom` 替代，标准库推荐以下方式实现类似效果：

| 替代方式                         | 示例代码                                                     | 特点                                   |
| :------------------------------- | :----------------------------------------------------------- | :------------------------------------- |
| **`.view.map(...).to(...)`**     | `List("London", "Paris").view.map(x => (x.length, x)).to(Map)` | **官方推荐**，懒加载中间步骤，清晰高效 |
| **`.iterator.map(...).to(...)`** | `List("London", "Paris").iterator.map(x => (x.length, x)).to(Map)` | 与视图类似，但迭代器不可重用           |

两种方式都能避免创建中间集合，性能与原先的 `breakOut` 相当。其中视图可以被多次重用，而迭代器在使用一次后就失效了[-14](https://stackoverflow.com/questions/68723881/conversion-of-breakout-use-iterator-or-view/68725937#68725937)。

> 对于需要一步完成类型转换的场景，优先使用 **`.view.map(...).to(...)`** 即可。

------

#### 💎 总结

- **版本支持**：`breakOut` 在 **Scala 2.8 ~ 2.12** 中可用，2.13 起被移除。
- **替代方案**：升级到 2.13+ 后，改用 **`.view.map(...).to(Collection)`**。
- **核心目标**：避免中间集合，提升转换性能与代码可读性。