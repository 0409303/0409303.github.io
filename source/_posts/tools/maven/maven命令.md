---
    title: maven命令
    categories: tools
    tags:
    creator: cjq
    create_time: 2021/01/27

---

#### 查看依赖jar包

```sh
mvn dependency:tree -Dverbose

mvn dependency:tree -Dverbose -Dincludes=org.apache.logging.log4j:log4j-api 
```



#### 打包

```shell
mvn clean -U package -pl analysis-tool -am -P dev -Dmaven.source.skip=true -Dmaven.test.skip=true
```

要注意的是-P 对大小写敏感



#### 重新生成iml文件

```
mvn idea:module
```



#### 参数

```
-e 详细错误描述
-X
```



## 问题汇总

#### the output path is not specified for module 'xxx'

Specify the output path in the Project Structure dalog

