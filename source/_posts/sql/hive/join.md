---
    title: hive-join
    categories: hive
    tags:
    creator: cjq
    create_time: 2021/05/14



---

## Join 类别

#### 1. inner join

#### 2. left/right join

#### 3. cross join

#### 4. left/right semi join 半开连接

LEFT SEMI JOIN:左半开连接会返回左边表的记录，前提是其记录对于右边表满足ON语句中的判定条件。对于常见的内连接（INNER JOIN），这是一个特殊的，优化了的情况。大多数的SQL方言会通过in.......exists结构来处理这种情况。

- 只会保留左/右表中能关联上的数据，且数量不会受另一张表影响而膨胀（即在原表中是几条数据，结果还是几条）

![](https://tva1.sinaimg.cn/large/008i3skNly1gqi7ne1000j314m0qewig.jpg)

总结：

对待右表中重复key的处理方式差异：因为 left semi join 是 in(keySet) 的关系，遇到右表重复记录，左表会跳过，而 join on 则会一直遍历。
left semi join 中最后 select 的结果只许出现左表，因为右表只有 join key 参与关联计算了，而 join on 默认是整个关系模型都参与计算了。

#### 5. 