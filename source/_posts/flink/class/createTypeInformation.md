---
    title: createTypeInfomation
    creator: cjq
    create_time: 2026/03/03
    tags:
      - flink
    categories:
      - [flink, class]

---





### ⚙️ 原理简述

- Flink 为许多常见 Scala 类型（如基本类型、集合、元组、case class 等）提供了内置的 `TypeInformation` 工厂。
- `createTypeInformation` 结合 `ClassTag` 和这些工厂，在编译期生成正确的 `TypeInformation`，避免了手动构造的繁琐和错误。

### ⚠️ 注意事项

- **必须导入**：如果忘记导入 `org.apache.flink.streaming.api.scala._`，编译器会报错“无法找到隐式值”，提示需要 `TypeInformation`。
- **自定义类型**：对于自定义的 case class 或类，只要其成员类型都有可用的 `TypeInformation`，`createTypeInformation` 也能自动组合生成。
- **Table API 中也有类似机制**，但通常通过导入 `org.apache.flink.table.api.bridge.scala._` 来提供。