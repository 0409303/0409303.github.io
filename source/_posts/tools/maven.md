---
	title:
	categories:
	tags:
---

#### 查看依赖jar包

mvn dependency:tree -Dverbose



#### 打包

```
mvn clean -U package -pl analysis-tool -am -P dev -Dmaven.source.skip=true -Dmaven.test.skip=true
```

要注意的是-P 对大小写敏感



