---
    title: flink窗口类
    categories: flink
    tags:
    creator: cjq
    create_time: 2021/01/22


---

- WindowAssigner
  - SlidingProcessingTimeWindows
  - BaseAlignedWindowAssigner
    - SlidingAlignedProcessingTimeWindows
  - TumblingEventTimeWindows
    - TumblingTimeWindows
  - MergingWindowAssigner
    - ProcessingTimeSessionWindows
    - DynamicProcessingTimeSessionWindows
    - DynamicEventTimeSessionWindows
    - EventTimeSessionWindows
  - TumblingProcessingTimeWindows
  - SlidingEventTimeWindows
    - SlidingTimeWindows
  - GlobalWindows



## WindowAssigner



### 1. SlidingProcessingTimeWindows

```java
/**
 * A {@link WindowAssigner} that windows elements into sliding windows based on the current
 * system time of the machine the operation is running on. Windows can possibly overlap.
 *
 * <p>For example, in order to window into windows of 1 minute, every 10 seconds:
 * <pre> {@code
 * DataStream<Tuple2<String, Integer>> in = ...;
 * KeyedStream<String, Tuple2<String, Integer>> keyed = in.keyBy(...);
 * WindowedStream<Tuple2<String, Integer>, String, TimeWindows> windowed =
 *   keyed.window(SlidingProcessingTimeWindows.of(Time.of(1, MINUTES), Time.of(10, SECONDS));
 * } </pre>
 */

/**
 * 基于flink处理时间的滑动窗口
 */
```

### 6. SlidingEventTimeWindows

```java
/**
 * A {@link WindowAssigner} that windows elements into sliding windows based on the timestamp of the
 * elements. Windows can possibly overlap.
 *
 * <p>For example, in order to window into windows of 1 minute, every 10 seconds:
 * <pre> {@code
 * DataStream<Tuple2<String, Integer>> in = ...;
 * KeyedStream<Tuple2<String, Integer>, String> keyed = in.keyBy(...);
 * WindowedStream<Tuple2<String, Integer>, String, TimeWindow> windowed =
 *   keyed.window(SlidingEventTimeWindows.of(Time.minutes(1), Time.seconds(10)));
 * } </pre>
 */

/**
 * 基于元素时间的滑动窗口
 */
```

#### 6.1 SlidingTimeWindows(废弃)



### 2. BaseAlignedWindowAssigner



#### 1.1 SlidingAlignedProcessingTimeWindows



### 5. TumblingProcessingTimeWindows



### 3. TumblingEventTimeWindows

```java
/**
 * A {@link WindowAssigner} that windows elements into windows based on the timestamp of the
 * elements. Windows cannot overlap.
 *
 * <p>For example, in order to window into windows of 1 minute:
 * <pre> {@code
 * DataStream<Tuple2<String, Integer>> in = ...;
 * KeyedStream<Tuple2<String, Integer>, String> keyed = in.keyBy(...);
 * WindowedStream<Tuple2<String, Integer>, String, TimeWindow> windowed =
 *   keyed.window(TumblingEventTimeWindows.of(Time.minutes(1)));
 * } </pre>
 */
```



#### 3.1 TumblingTimeWindows(废弃)

```java
/**
 * A {@link WindowAssigner} that windows elements into windows based on the timestamp of the
 * elements. Windows cannot overlap.
 *
 * @deprecated Please use {@link TumblingEventTimeWindows}.
 */
```



### 4. MergingWindowAssigner



#### 4.1 ProcessingTimeSessionWindows



#### 4.2 DynamicProcessingTimeSessionWindows



#### 4.3 DynamicEventTimeSessionWindows



#### 4.4 EventTimeSessionWindows



### 7. GlobalWindows



## 总结

### 适用场景

1. **滑动窗口**

- 每条数据会发送到多个滑动窗口中，即在最终的输出中，一条数据要被统计多次
- 适合统计据当前时间往前一段时间内的数据汇总

2. **滚动窗口**

- 每条数据只会在一个滚动窗口中
- 适合对数据进行简单聚合后，再次聚合的场景
- 适合输出明细，不做聚合的场景，比如join后直接输出



### 占用内存比较

1. **滑动窗口**
   - *理论上一条数据会复制到多个窗口，被复制几次，占用内存就会扩大几倍，但不清楚是否有优化，比如只复制数据的引用？*
   - 
2. **滚动窗口**



### 关于时间点

#### 窗口的开始时间

1. 首先要明确的是，开始时间只与system time和offset参数相关，与程序开始运行时间无关
2. 比如设置了窗口size是1h，那么在

#### 窗口的结束时间