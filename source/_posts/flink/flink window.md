---
    title: flink window
    categories: flink
    tags:
    creator: cjq
    create_time: 2021/01/29


---



# flink1.6

[官网链接](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/stream/operators/windows.html)

**Keyed Windows**

```
stream
       .keyBy(...)               <-  keyed versus non-keyed windows
       .window(...)              <-  required: "assigner"
      [.trigger(...)]            <-  optional: "trigger" (else default trigger)
      [.evictor(...)]            <-  optional: "evictor" (else no evictor)
      [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
      [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
       .reduce/aggregate/fold/apply()      <-  required: "function"
      [.getSideOutput(...)]      <-  optional: "output tag"
```

**Non-Keyed Windows**

```
stream
       .windowAll(...)           <-  required: "assigner"
      [.trigger(...)]            <-  optional: "trigger" (else default trigger)
      [.evictor(...)]            <-  optional: "evictor" (else no evictor)
      [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
      [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
       .reduce/aggregate/fold/apply()      <-  required: "function"
      [.getSideOutput(...)]      <-  optional: "output tag"
```



## 窗口生命周期

### window



### trigger

The function will contain the computation to be applied to the contents of the window, while the `Trigger` specifies the conditions under which the window is considered ready for the function to be applied. A triggering policy might be something like “when the number of elements in the window is more than 4”, or “when the watermark passes the end of the window”. A trigger can also decide to purge a window’s contents any time between its creation and removal. Purging in this case only refers to the elements in the window, and *not* the window metadata. This means that new data can still be added to that window.

function包含了窗口中数据的计算逻辑，trigger指定了在哪些条件下会调用这些function

### Evictor

Apart from the above, you can specify an `Evictor` (see [Evictors](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/stream/operators/windows.html#evictors)) which will be able to remove elements from the window after the trigger fires and before and/or after the function is applied.